---
name: seogeoaeo
description: Research competitors, keywords, ads and AI citations, and check a public website's SEO, answer readiness and AI visibility with the seogeoaeo.ai API, then fix what it finds and re-run to verify. Use when the user asks to find competitors, size up a domain, see a site's backlinks, see what a competitor ranks for or the keywords it wins that they miss, get search volume, difficulty and cost per click for keywords, see the Google, Facebook, Instagram or LinkedIn ads competitors run, see which pages Google AI Overviews and ChatGPT cite, audit or improve a site or page for search engines or AI assistants (indexing, robots.txt, redirects, titles, headings, structured data, content clarity, authorship, rendering, Core Web Vitals, sitemaps, hreflang), sample how ChatGPT, Perplexity and Gemini answer about their brand, or generate JSON-LD.
---

# seogeoaeo.ai tools

seogeoaeo.ai runs single-purpose checks on public websites. Each run costs credits from the user's workspace and returns findings with a severity, why each matters, and how to fix it, plus a prioritized list of changes an agent can apply and verify.

## Setup

1. The user creates an API key at https://seogeoaeo.ai/api-keys. Keys start with `sga_`.
2. Read the key from the `SEOGEOAEO_API_KEY` environment variable. Never print it, write it to files, or paste it into chat.
3. If the `seogeoaeo` MCP server is connected, call its tools directly and skip the curl calls below.

## Running a tool

```bash
curl -X POST https://seogeoaeo.ai/api/v1/tools/indexing-checker/runs \
  -H "Authorization: Bearer $SEOGEOAEO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"input":{"url":"https://example.com"}}'
```

The response is `{ "run": { "id", "status", "creditsCharged", "result", "improvementTips", ... } }`. Read `run.result` for the raw output (most tools return a `findings` array ordered by severity) and `run.improvementTips` for what to change.

Rules:

- Check the balance before a batch: `curl -H "Authorization: Bearer $SEOGEOAEO_API_KEY" https://seogeoaeo.ai/api/v1/credits`.
- Pass `"maxCredits"` when the user set a budget; the run is refused with `max_credits_exceeded` instead of charged.
- Allow four minutes for AI answer, brand narrative and fan-out reports. Some providers take over two minutes to answer. If a request times out, retry with the same idempotencyKey. Check improvementTips.unchecked for any provider data that could not be retrieved.
- When retrying after a timeout or network error, send the same `"idempotencyKey"` so the run is not charged twice. A key is remembered while its run is among the workspace's latest 1,000.
- Tools accept public http(s) URLs only. Do not send localhost, private IPs, or URLs with credentials.
- Fields described as lists are newline-separated strings, not JSON arrays.
- On `429 rate_limited`, wait the seconds in the `Retry-After` header before retrying. Each tool allows 20 runs per 10 minutes per workspace, and each key 120 requests per minute.
- On `409 run_in_progress`, a run with that idempotencyKey is still running. Wait, then fetch `GET /runs/{runId}` with the `runId` from the error.
- On `402 insufficient_credits`, stop and tell the user to choose a plan or add top-up credits at https://seogeoaeo.ai/account. Top-up credits (`locked` in the balance) are spendable only while a plan is active.

## Improving a site

Every succeeded run returns `run.improvementTips` next to `run.result`:

- `status`: `changes` when there are tips, `clean` when the tool checked everything and found nothing to change, and `incomplete` when part of the analysis could not run.
- `summary`: a plain-text checklist of every change, ordered by priority.
- `tips`: `[{ id, priority, title, action, why, detail, affected, fixIn? }]`, ordered high, medium, low. `action` is the change to make, `affected` lists the URLs, headers, or items it applies to, and `id` names one kind of change and stays the same across runs. `fixIn` is `dns` or `external` when the change is made outside the site's code, in DNS or another service.
- `unchecked`: `[{ id, title, detail, action }]`, the parts of the analysis that could not run, such as a data provider outage or an input with nothing to analyze. While it is not empty, the result cannot show that the site is clean.
- `rerunVerifies`: `true` when running the tool again with the same input shows whether the changes worked. `false` for generators and for runs on pasted input or search data, whose tips come back on every run with the same input.
- `verify`: how to confirm the changes worked.

Passing checks, notes about the input, and caveats are left out of `tips`. Failed runs, and runs whose result is no longer available, return `improvementTips: null`.
Brand Narrative's `conflicts` are model-flagged candidates for human review, not verified contradictions. Quote matching proves that the text exists, not that the statements disagree. These candidates remain report notes and must not trigger automatic site edits. Product and LocalBusiness markup are conditional on the page's purpose; a missing type alone is not a required fix. llms.txt is optional and is not a Google search or AI visibility requirement.

When the user asks to check a site and improve it:

1. Check the credit balance with the check-credits MCP tool or `GET /credits`. Add up the price of each planned run and confirm the total with the user before running more than the core checks.
2. Start with one website assessment: the run-assessment MCP tool, or `POST /assessments` with `{ "input": { "url": "https://example.com/", "pages": [] } }` and up to two key pages. It runs the core checks, `robots-checker`, `redirects-headers-checker` once on the origin (20 credits) and `indexing-checker`, `on-page-seo-checker`, `schema-checker`, `readability-analyzer`, `citability-analyzer`, `content-freshness-checker`, `author-eeat-checker` on each page (47 credits per page), so one page costs 67 credits. It returns `scores` from 0 to 100 for SEO readiness, AEO readiness and AI access with the PageSpeed Insights bands, one combined `improvementTips` list, and `next`. When `status` is `in_progress`, continue it with its id (run-assessment with `id`, or `POST /assessments/{id}/continue`). To answer a focused question, run the single tool instead.
3. Add a conditional check only when its condition holds:
   - When the site has more than a handful of pages: `xml-sitemap-validator`, `orphan-page-finder`, `broken-internal-link-finder`.
   - When pages depend on JavaScript, are very large, or differ on mobile: `ai-crawler-view`, `mobile-parity-checker`, `page-size-checker`.
   - When speed or images matter for the page: `core-web-vitals-snapshot`, `image-seo-checker`.
   - When the page sells a product: `product-schema-checker`.
   - When the business serves customers at a location: `local-business-checker`.
   - When the site has language or country versions: `hreflang-checker`.
   - When brand recognition, the search result favicon or link previews matter: `brand-entity-checker`, `favicon-checker`, `open-graph-checker`.
   - When the user asks whether AI browsers and agents can use the site: `ai-agent-readiness`.
   - When changed URLs should be announced to Bing and other IndexNow engines: `indexnow-checker`.
4. AI answer sampling: `ai-answer-visibility` costs 120 credits per run, `brand-narrative-check` costs 150 credits per run, `fanout-coverage-checker` costs 95 credits per run. Run these only when the user asks how AI assistants answer questions about the market or describe the brand, and agree on the exact questions first. Results are sampled observations that can change between runs; never report them as verified site improvements.
5. Run input-driven tools only when there is input for them, such as a target query, a keyword list, a log file, or markup to generate: `competitor-finder`, `domain-overview`, `backlink-finder`, `competitor-keywords`, `keyword-gap`, `keyword-research`, `keyword-metrics`, `competitor-google-ads`, `meta-ad-finder`, `linkedin-ad-finder`, `ai-citation-finder`, `ai-question-finder`, `http-status-bulk-checker`, `keyword-ideas`, `keyword-clusterer`, `keyword-deduplicator`, `schema-markup-generator`, `organization-entity-markup-builder`, `faq-schema-builder`, `breadcrumb-schema-builder`, `llms-txt-generator`, `ai-bot-log-analyzer`, `question-finder`.
6. Work through each run's tips from high to low. Find the code, template, config, or content that produces each affected URL or item, and make the change `action` describes. Do not invent facts, reviews, prices, or contact details to satisfy a tip.
7. When a tip has `fixIn`, or otherwise needs a change outside the codebase such as DNS, CDN, or hosting settings, tell the user exactly what to change.
8. Deploy, then check again. Tools only fetch public URLs, so they see a change once it is live. For an assessment, run a new one on the same URL: it is compared with the previous one automatically, and `progress` gives `percentFixed` with the counts of `fixed`, `stillPresent` and `new` issues. For a single tool, run the same tool with the same input; a tip whose `id` no longer appears is fixed. Repeat until `status` is `clean` or the remaining tips need the user.
9. When `rerunVerifies` is `false`, apply the tips once and do not re-run for them, because the same input returns the same tips. Where a tip names a checker such as `schema-checker`, run that checker on the live URL instead.
10. When `status` is `incomplete`, follow each `unchecked` item before reporting the site as clean. A provider outage clears on a later run; an input problem needs a different input.
11. Report the scores before and after, `progress.percentFixed`, what remains and why, what could not be checked, and the credits spent. Link the assessment's `reportUrl` so the user can see the report.

People who prefer the browser can run the same assessment, with the same scores and comparison, at https://seogeoaeo.ai/check-website. AI-referral filters are free at https://seogeoaeo.ai/guides/ai-referrals, and owner-data measurement guidance is at https://seogeoaeo.ai/guides/search-performance.

## Choosing a tool

Start with the narrowest tool that answers the question. For a general site check, run one website assessment (run-assessment or `POST /assessments`). For a question about one page, start with `indexing-checker` and `on-page-seo-checker`. For whether AI crawlers can read a site, run `robots-checker` and `ai-crawler-view`. For how AI assistants describe or recommend a brand, agree on the questions with the user, then run `ai-answer-visibility` or `brand-narrative-check`.

### SEO

| Tool | Credits | Inputs | What it does |
| --- | --- | --- | --- |
| `competitor-finder` | 38 | url (required), market | See which sites compete with yours in Google, so you know whose keywords to study. |
| `domain-overview` | 50 | url (required), market | Size up any website, yours or a competitor's, by its Google rankings and its links before you decide what to copy or beat. |
| `backlink-finder` | 70 | url (required) | See which sites link to a competitor, or to you, so you know who to ask for a link and which links are worth matching. |
| `competitor-keywords` | 35 | url (required), market | See what a competitor ranks for in Google, then pick blog topics and ad keywords from it. |
| `keyword-gap` | 85 | url (required), competitor (required), market | See the keywords a competitor wins in Google that your site misses, then pick blog topics and ad keywords from them. |
| `keyword-research` | 34 | keyword (required), market | Turn one keyword or topic into a list of related Google searches with the data to choose blog topics and ad keywords. |
| `keyword-metrics` | 33 | keywords (required), market | Rank a keyword list you already have by search volume, difficulty and ad cost, so you know which to write about or bid on first. |
| `competitor-google-ads` | 15 | url (required), market | See what a competitor says in its Google ads, and which ads it keeps paying for, so you can write stronger ones. |
| `meta-ad-finder` | 5 | query (required), mode, market | See the Facebook and Instagram ads your competitors keep running, and the wording and offers behind them. |
| `linkedin-ad-finder` | 5 | company (required), market | See what a company says in its LinkedIn ads, which ones it keeps running, and which ad types it relies on. |
| `indexing-checker` | 8 | url (required) | See whether Google and Bing may crawl, index and quote a page in search and AI answers, which signal blocks it, and the exact tag or header to change. |
| `robots-checker` | 10 | url (required), path, userAgent | See every robots.txt rule that blocks search or AI crawlers, the rule that matches a given path, and the exact line to change. |
| `redirects-headers-checker` | 10 | url (required) | See every redirect hop, duplicate host version and response header that costs crawl budget or blocks indexing, with the server rule to change. |
| `on-page-seo-checker` | 8 | url (required), query | See the title, description and heading changes a page needs, how its snippet clips in search, and which query words it is missing. |
| `xml-sitemap-validator` | 10 | url (required) | See whether search engines can read your sitemap, how many entries it has, and which URLs or dates are wrong. |
| `http-status-bulk-checker` | 20 | urls (required) | See status, redirect target, and timing for this URL list. Page bodies are not fetched. |
| `broken-internal-link-finder` | 15 | url (required) | Find links on one page that lead to broken pages on the same site. |
| `open-graph-checker` | 5 | url (required) | See the preview LinkedIn, Facebook, Slack and X build, with image size, format and dimension checks against each platform’s limits. |
| `readability-analyzer` | 5 | url, text | See reading level, long-sentence issues, and the sentences most worth rewriting. |
| `keyword-ideas` | 15 | query (required), market | See the keyword suggestions Google shows for a seed in your market. |
| `keyword-clusterer` | 3 | keywords (required) | Group a keyword list into topic clusters, with a suggested parent page path for each group. |
| `keyword-deduplicator` | 3 | keywords (required) | See unique keywords after normalization, plus merged duplicates and conflicting spellings. |
| `core-web-vitals-snapshot` | 20 | url (required), strategy | Review field measurements and lab diagnostics separately for one public URL, with clear gaps when data is unavailable. |
| `page-size-checker` | 5 | url (required) | Know whether Googlebot sees the whole page, and exactly which links, JSON-LD and text fall past the 2 MB cutoff when it does not. |
| `hreflang-checker` | 10 | url (required) | Find the invalid codes, missing return links, noindex and canonical conflicts that make Google ignore your language versions. |
| `favicon-checker` | 5 | url (required) | Check whether your favicon and site name meet the requirements Google uses to show them in search results. |
| `indexnow-checker` | 5 | url (required), key, keyLocation, urls | Notify participating engines about changed URLs and see whether the submission was accepted. Crawling and indexing timing are not guaranteed. |
| `mobile-parity-checker` | 5 | url (required) | Find content, links and markup missing from the mobile version of a page, which is the version Google indexes. |
| `product-schema-checker` | 5 | url (required) | Find the Product markup fields missing or wrong for Google's price, availability and rating rich results. |
| `local-business-checker` | 5 | url (required) | Check that your LocalBusiness markup is complete and that its address and phone match what the page shows. |
| `image-seo-checker` | 10 | url (required) | Find images missing alt text or dimensions, broken image files, and heavy or oversized images slowing the page down. |
| `orphan-page-finder` | 20 | url (required) | Find pages nothing links to, pages buried too deep, and sitemap entries that fail, within one 150-page crawl. |

### AEO

| Tool | Credits | Inputs | What it does |
| --- | --- | --- | --- |
| `schema-checker` | 8 | url, jsonld | See which JSON-LD blocks fail to parse, which required and recommended properties are missing, and which types the page is missing. |
| `schema-markup-generator` | 3 | type (required), name, headline, url, description, image, author, datePublished, publisher, dateModified, logo, telephone, price, priceCurrency, ratingValue, ratingCount, applicationCategory, operatingSystem, startDate, endDate, location, streetAddress, addressLocality, addressRegion, postalCode, addressCountry, thumbnailUrl, uploadDate, faqs, breadcrumbs, steps, ingredients | Get valid JSON-LD for a documented type from the fields you supply. Nothing is fetched. |
| `organization-entity-markup-builder` | 3 | name (required), legalName, alternateName, url, logo, description, email, telephone, sameAs, foundingDate, taxID, vatID, streetAddress, addressLocality, addressRegion, postalCode, addressCountry, contactType | Get Organization JSON-LD from the details you supply. Nothing is fetched. |
| `faq-schema-builder` | 3 | faqs (required), name, url | Get FAQPage JSON-LD from the questions you supply. Nothing is fetched. |
| `breadcrumb-schema-builder` | 3 | breadcrumbs, url | Get BreadcrumbList JSON-LD from a trail or a page URL path. Nothing is fetched. |
| `content-freshness-checker` | 5 | url (required) | Find missing or conflicting date signals and review whether dated claims need a substantive update. |
| `author-eeat-checker` | 8 | url (required) | Check whether a page clearly shows who wrote it and who stands behind it, in visible text and markup. |
| `question-finder` | 15 | query (required), market, url | See the questions Google shows for a topic and which ones your page leaves unanswered. |

### GEO

| Tool | Credits | Inputs | What it does |
| --- | --- | --- | --- |
| `ai-citation-finder` | 160 | mode, query (required), platform, market | Find the pages AI answers quote, so you know what to write, what to keep current, and which sites to get mentioned on. |
| `ai-question-finder` | 160 | mode, query (required), platform, market | Find the questions AI answers respond to, so you know what to write and which pages the answers lean on. |
| `ai-agent-readiness` | 10 | url (required), llmsTxt | Check whether AI browsers and agents can read and use a site, with each page attribute, llms.txt line or discovery file to fix. |
| `llms-txt-generator` | 15 | url (required), site | Get a draft llms.txt built only from pages this run could fetch, ready to edit and publish. |
| `citability-analyzer` | 5 | url (required) | Find passages that may need clearer answers or context, with suggestions to review before editing. |
| `ai-bot-log-analyzer` | 8 | log (required) | See what GPTBot, ClaudeBot, PerplexityBot and ChatGPT-User actually fetch from your site, and catch impostors. |
| `ai-crawler-view` | 8 | url (required) | Find the content AI crawlers miss because it loads with JavaScript. |
| `brand-entity-checker` | 8 | url (required), brand | Check whether your brand name, logo, Organization markup and profiles consistently tie your brand to your site. |
| `fanout-coverage-checker` | 95 | query (required), url (required) | Find the related searches AI engines run behind a query that your page does not answer yet. |
| `ai-answer-visibility` | 120 | url (required), prompt (required), brand | Sample ChatGPT, Perplexity and Gemini answers to one question and see whether they mention or cite your brand, and who they name instead. |
| `brand-narrative-check` | 150 | url (required), brand | See what ChatGPT, Perplexity and Gemini tell people about your brand, with quoted evidence for potential discrepancies. |

## Reference

- Full input schemas: https://seogeoaeo.ai/api/v1/tools (JSON Schema per tool, no key needed)
- OpenAPI: https://seogeoaeo.ai/openapi.json
- Every tool's inputs, limits, and non-goals: https://seogeoaeo.ai/llms-full.txt
- Setup guide: https://seogeoaeo.ai/agents
