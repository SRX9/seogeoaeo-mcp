<p align="center">
  <img src="assets/logo-512.png" alt="seogeoaeo.ai" width="96" height="96">
</p>

# seogeoaeo.ai for AI agents

Research competitors, keywords, ads and AI citations, and check any public
website for SEO, AEO and GEO readiness, from your coding agent.
This repo packages the [seogeoaeo.ai](https://seogeoaeo.ai) MCP server, REST API
and agent skill so Claude Code, Cursor, Codex, VS Code and any other MCP client
can install them in one step.

Each tool does one job. A site check returns findings with a severity, why each
one matters and how to fix it, plus a prioritized list of changes your agent can
apply. Re-run the same tool afterwards to verify the fix. A research tool returns
the data: competitors, keywords, ads or the pages AI answers cite.

- MCP endpoint: `https://seogeoaeo.ai/api/mcp` (stateless Streamable HTTP, POST only)
- REST API: `https://seogeoaeo.ai/api/v1`, described by [`/openapi.json`](https://seogeoaeo.ai/openapi.json)
- Docs for agents: [`/agents`](https://seogeoaeo.ai/agents), [`/llms.txt`](https://seogeoaeo.ai/llms.txt), [`/llms-full.txt`](https://seogeoaeo.ai/llms-full.txt)

## 1. Get an API key

Sign in at [seogeoaeo.ai](https://seogeoaeo.ai), then create a key at
[seogeoaeo.ai/api-keys](https://seogeoaeo.ai/api-keys). Keys start with `sga_`.
Runs spend credits from your workspace plan, so a plan must be active.

Put the key in an environment variable instead of pasting it into config files:

```bash
export SEOGEOAEO_API_KEY=sga_your_key_here
```

## 2. Install

### Claude Code

```bash
claude mcp add --transport http --scope user seogeoaeo https://seogeoaeo.ai/api/mcp \
  --header "Authorization: Bearer $SEOGEOAEO_API_KEY"
```

Or install the plugin, which adds the MCP server and the skill together:

```bash
claude plugin marketplace add seogeoaeo/seogeoaeo-mcp
claude plugin install seogeoaeo@seogeoaeo
```

The plugin reads `SEOGEOAEO_API_KEY` from your environment.

### Cursor

Add this to `~/.cursor/mcp.json` (or `.cursor/mcp.json` in a project):

```json
{
  "mcpServers": {
    "seogeoaeo": {
      "url": "https://seogeoaeo.ai/api/mcp",
      "headers": { "Authorization": "Bearer ${env:SEOGEOAEO_API_KEY}" }
    }
  }
}
```

The repo also ships a Cursor plugin manifest in [`.cursor-plugin/`](.cursor-plugin/plugin.json).

### VS Code

Add this to `.vscode/mcp.json`:

```json
{
  "servers": {
    "seogeoaeo": {
      "type": "http",
      "url": "https://seogeoaeo.ai/api/mcp",
      "headers": { "Authorization": "Bearer ${env:SEOGEOAEO_API_KEY}" }
    }
  }
}
```

### Codex

Add this to `~/.codex/config.toml`:

```toml
[mcp_servers.seogeoaeo]
url = "https://seogeoaeo.ai/api/mcp"
bearer_token_env_var = "SEOGEOAEO_API_KEY"
```

### Claude Desktop and other stdio-only clients

Bridge the remote server with `mcp-remote`:

```json
{
  "mcpServers": {
    "seogeoaeo": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://seogeoaeo.ai/api/mcp",
        "--header",
        "Authorization:${AUTH_HEADER}"
      ],
      "env": { "AUTH_HEADER": "Bearer sga_your_key_here" }
    }
  }
}
```

### Skill only

If you only want the agent skill, copy [`skills/seogeoaeo`](skills/seogeoaeo/SKILL.md)
into your agent's skills directory. It teaches the agent to call the REST API with
`curl`, check the credit balance first, retry safely, pick the right research or
check tool and apply the returned fixes.
The always-current copy is served at
[`seogeoaeo.ai/skills/seogeoaeo/SKILL.md`](https://seogeoaeo.ai/skills/seogeoaeo/SKILL.md).

## What the agent can do

51 tools in two groups, each its own MCP tool named by its slug.

**Research** answers questions about the market: who your competitors are, what
they rank for and which keywords you miss, keyword search volume, difficulty and
cost per click, the Google, Facebook, Instagram and LinkedIn ads competitors keep
running, and which pages Google AI Overviews and ChatGPT cite for a topic.

**Site checks** test your own pages. Start with one website assessment
(`run-assessment`). It runs the core checks, scores SEO readiness, AEO readiness
and AI access from 0 to 100 like PageSpeed Insights, and returns one fix list
ordered by priority. Run it again after deploying to see the percent of earlier
issues fixed. For a focused question, run a single check.

Prices below are from October 2026. Each MCP tool description carries the live
price.

### Research: competitors, keywords, ads and AI citations

**Competitor research**

- `competitor-finder` (38 credits): Competitor Finder
- `domain-overview` (50 credits): Domain Overview
- `backlink-finder` (70 credits): Backlink Finder
- `competitor-keywords` (35 credits): Competitor Keywords
- `keyword-gap` (85 credits): Keyword Gap

**Keyword research**

- `keyword-research` (34 credits): Keyword Research
- `keyword-metrics` (33 credits): Keyword Metrics

**Competitor ads**

- `competitor-google-ads` (15 credits): Competitor Google Ads
- `meta-ad-finder` (5 credits): Meta Ad Finder
- `linkedin-ad-finder` (5 credits): LinkedIn Ad Finder

**AI answer citations**

- `ai-citation-finder` (160 credits): AI Citation Finder
- `ai-question-finder` (160 credits): AI Question Finder

### Site checks: SEO

**Technical page check**

- `indexing-checker` (8 credits): Indexing & Canonical Checker
- `robots-checker` (10 credits): Robots.txt & AI Crawler Checker
- `redirects-headers-checker` (10 credits): Redirects, Headers & Host Checker

**Site structure**

- `orphan-page-finder` (20 credits): Orphan Page Finder
- `broken-internal-link-finder` (15 credits): Broken Internal Link Finder
- `xml-sitemap-validator` (10 credits): XML Sitemap Validator
- `indexnow-checker` (5 credits): IndexNow Checker & Submitter

**On-page SEO**

- `on-page-seo-checker` (8 credits): On-Page SEO Checker
- `image-seo-checker` (10 credits): Image SEO Checker

**Rendering and mobile**

- `ai-crawler-view` (8 credits): AI Crawler View
- `mobile-parity-checker` (5 credits): Mobile Parity Checker
- `page-size-checker` (5 credits): Page Size & 2 MB Limit Checker

**Performance**

- `core-web-vitals-snapshot` (20 credits): Core Web Vitals Snapshot

**Keyword lists**

- `keyword-deduplicator` (3 credits): Keyword Deduplicator
- `keyword-clusterer` (3 credits): Keyword Clusterer

**International SEO**

- `hreflang-checker` (10 credits): Hreflang Checker

**Bulk status checks**

- `http-status-bulk-checker` (20 credits): HTTP Status Bulk Checker

**Social and search previews**

- `open-graph-checker` (5 credits): Open Graph & Social Card Checker
- `favicon-checker` (5 credits): Favicon & Site Name Checker

### Site checks: AEO

**Structured data check**

- `schema-checker` (8 credits): Schema Markup Checker
- `product-schema-checker` (5 credits): Product Schema Checker
- `local-business-checker` (5 credits): Local Business Checker

**Schema builder**

- `schema-markup-generator` (3 credits): Schema Markup Generator
- `organization-entity-markup-builder` (3 credits): Organization Entity Markup Builder
- `faq-schema-builder` (3 credits): FAQ Schema Builder
- `breadcrumb-schema-builder` (3 credits): Breadcrumb Schema Builder

**Content and answer readiness**

- `citability-analyzer` (5 credits): Citability Analyzer
- `readability-analyzer` (5 credits): Readability Analyzer
- `content-freshness-checker` (5 credits): Content Freshness Checker
- `author-eeat-checker` (8 credits): Author & E-E-A-T Checker

**Questions and topic coverage**

- `question-finder` (15 credits): Question Finder
- `keyword-ideas` (15 credits): Keyword Ideas
- `fanout-coverage-checker` (95 credits): Fan-out Coverage Checker

### Site checks: GEO

**Brand and AI visibility**

- `brand-entity-checker` (8 credits): Brand Entity Checker
- `ai-answer-visibility` (120 credits): AI Answer Visibility
- `brand-narrative-check` (150 credits): Brand Narrative Check

**AI-agent utilities**

- `ai-agent-readiness` (10 credits): AI Agent Readiness Checker
- `llms-txt-generator` (15 credits): llms.txt Generator

**AI bot logs**

- `ai-bot-log-analyzer` (8 credits): AI Bot Log Analyzer

Helper tools: `run-assessment`, `get-assessment`, `check-credits`, `get-run`.

## Pricing

Each tool is priced in credits, and the price is in the tool's description. Most
site checks cost 3 to 20 credits. Research tools cost 5 to 85 credits, except
`ai-citation-finder` and `ai-question-finder` at 160. The three tools that
sample AI answers (`fanout-coverage-checker` 95, `ai-answer-visibility` 120,
`brand-narrative-check` 150) call paid model providers, so the agent runs them
only when you ask. A one-page website assessment costs 67 credits. Failed runs
are refunded.

| Plan | Price | Credits |
|------|-------|---------|
| Indie | $29/month | 2,000 |
| Startup | $69/month | 5,000 |
| Scale | $199/month | 22,000 |
| Enterprise | $499/month | 130,000 |

See [seogeoaeo.ai/pricing](https://seogeoaeo.ai/pricing) for current prices and
top-up packs. The agent can pass `maxCredits` on any run to refuse a price above
what you expect, and `idempotencyKey` so a retry is never charged twice.

## Limits and safety

- Tools accept public `http(s)` URLs only. Localhost, private IPs and URLs with credentials are rejected.
- Each tool allows 20 runs per 10 minutes per workspace and each key 120 requests per minute. A `429` carries `Retry-After`.
- Keys are stored as SHA-256 hashes. Revoke a key at any time at [seogeoaeo.ai/api-keys](https://seogeoaeo.ai/api-keys). Each workspace can hold 10 active keys.
- Run results are kept for the latest 1,000 runs per workspace.

## Repo layout

| Path | Purpose |
|------|---------|
| `server.json` | Official MCP Registry listing |
| `.mcp.json` | Claude Code and Codex plugin MCP config |
| `mcp.json` | Cursor plugin MCP config |
| `.claude-plugin/` | Claude Code plugin and marketplace manifests |
| `.cursor-plugin/` | Cursor plugin manifest |
| `.codex-plugin/`, `.agents/plugins/` | Codex plugin manifest and marketplace |
| `skills/seogeoaeo/SKILL.md` | Agent skill |
| `assets/` | Logo (SVG and 512px PNG) |
| `glama.json` | Glama ownership claim |
| `SUBMISSIONS.md` | Where to submit and what to paste |

This repo holds configuration only. The service itself is hosted at
[seogeoaeo.ai](https://seogeoaeo.ai).

## Support

Questions or problems: [seogeoaeo.ai/contact](https://seogeoaeo.ai/contact).

## License

[MIT](LICENSE) for the contents of this repo. Use of the hosted service is
covered by the [terms](https://seogeoaeo.ai/terms) and
[privacy policy](https://seogeoaeo.ai/privacy).
