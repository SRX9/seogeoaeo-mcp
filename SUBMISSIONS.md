# Where to submit seogeoaeo.ai

Submit in this order. Every listing points back to this repo and to
`https://seogeoaeo.ai/api/mcp`. Submission forms change often, so check each
site's current form before you paste.

## Copy-paste fields

| Field | Value |
|-------|-------|
| Name | seogeoaeo.ai |
| Slug / server name | `seogeoaeo` (registry: `ai.seogeoaeo/mcp`) |
| Tagline (98 chars) | Research competitors, keywords and ads, and check any site for SEO, AEO and GEO with ranked fixes. |
| Short description | SEO research and site checks |
| Category | Developer Tools (or Marketing / SEO where offered) |
| Keywords | seo, aeo, geo, ai-search, competitor-research, keyword-research, backlinks, ad-research, ai-citations, llms-txt, structured-data, robots-txt, core-web-vitals, mcp |
| MCP endpoint | `https://seogeoaeo.ai/api/mcp` |
| Transport | Streamable HTTP, stateless, POST only |
| Auth | API key sent as `Authorization: Bearer sga_...` (no OAuth yet) |
| Get a key | https://seogeoaeo.ai/api-keys |
| Website / docs | https://seogeoaeo.ai/agents |
| OpenAPI | https://seogeoaeo.ai/openapi.json |
| llms.txt | https://seogeoaeo.ai/llms.txt |
| Skill | https://seogeoaeo.ai/skills/seogeoaeo/SKILL.md |
| Privacy | https://seogeoaeo.ai/privacy |
| Terms | https://seogeoaeo.ai/terms |
| Support | https://seogeoaeo.ai/contact |
| Repository | https://github.com/SRX9/seogeoaeo-mcp |
| Logo | `assets/logo.svg`, `assets/logo-512.png`, `assets/logo-400.png` |
| Pricing | Paid. Plans from $29/month for 2,000 credits; each tool costs 3 to 160 credits. |

Long description:

> Research competitors, keywords, ads and AI citations, and run SEO, AEO and GEO
> checks on any public website, then apply the ranked fixes and re-run to verify.
> Find competitors, see their backlinks, rankings and keyword gaps, get search
> volume and difficulty, see the Google, Facebook, Instagram and LinkedIn ads they
> run, and see which pages AI Overviews and ChatGPT cite. Site checks cover
> indexing, robots.txt and AI crawlers, structured data, rendering, Core Web
> Vitals, llms.txt and AI answer visibility.

## 1. Official MCP Registry

Feeds GitHub's MCP registry, VS Code's MCP gallery and several directories that
sync from it, so do this first.

- File: [`server.json`](server.json)
- Tool: `mcp-publisher` from https://github.com/modelcontextprotocol/registry
- Namespace `ai.seogeoaeo/*` needs domain proof: `mcp-publisher login dns`
  (TXT record on seogeoaeo.ai) or `mcp-publisher login http` (serve
  `/.well-known/mcp-registry-auth`). The quicker route is renaming the server to
  `io.github.<github-user-or-org>/seogeoaeo` and using `mcp-publisher login github`.
- Then `mcp-publisher publish`. Bump `version` for every later publish.

## 2. Claude

- **Claude Code plugin marketplace (works today):** this repo is the
  marketplace. Users run `claude plugin marketplace add SRX9/seogeoaeo-mcp`.
  Files: [`.claude-plugin/`](.claude-plugin/), [`.mcp.json`](.mcp.json),
  [`skills/`](skills/).
- **Anthropic's plugin directory and connectors directory:** submit through
  Anthropic's forms. The connectors directory requires OAuth, so it waits until
  `/api/mcp` supports OAuth.

## 3. Cursor Marketplace

- Files: [`.cursor-plugin/plugin.json`](.cursor-plugin/plugin.json),
  [`mcp.json`](mcp.json), [`skills/`](skills/), `assets/logo.svg`.
- Submit the public repo URL at https://cursor.com/marketplace.

## 4. OpenAI (Codex plugins, ChatGPT apps)

- Files: [`.codex-plugin/plugin.json`](.codex-plugin/plugin.json),
  [`.agents/plugins/marketplace.json`](.agents/plugins/marketplace.json),
  [`.mcp.json`](.mcp.json).
- Check both files against OpenAI's current plugin docs first; they were written
  from public examples and not validated.
- ChatGPT apps expect OAuth.

## 5. Smithery

- Add a hosted server at https://smithery.ai/new with the endpoint URL. Smithery
  scans tools by connecting, so give it a test key or wait for OAuth.

## 6. Glama

- Submit the repo at https://glama.ai/mcp/servers. [`glama.json`](glama.json)
  claims ownership; its `maintainers` must list the GitHub usernames that
  should control the listing.

## 7. Directories (PR or form, no extra files)

- **Cline MCP Marketplace:** open an issue at
  https://github.com/cline/mcp-marketplace with the repo URL and
  `assets/logo-400.png`.
- **awesome-mcp-servers:** PR a one-line entry to
  https://github.com/punkpeye/awesome-mcp-servers under "Marketing" or
  "Search & Data Extraction". Remote server, so mark it ☁️.
- **PulseMCP:** https://www.pulsemcp.com/submit (also syncs from the official
  registry).
- **mcp.so:** submit the repo URL.
- **LobeHub MCP marketplace:** https://lobehub.com/mcp, submit the repo URL.

## 8. Skill directories

- The skill is at [`skills/seogeoaeo/SKILL.md`](skills/seogeoaeo/SKILL.md).
  skills.sh lists skills that people install with
  `npx skills add SRX9/seogeoaeo-mcp`, so share that command on `/agents`
  and in launch posts.

## Keeping this repo current

`skills/seogeoaeo/SKILL.md` and the tool list in `README.md` are copies of what
the seogeoaeo.ai registry generates. When tools or prices change, copy
https://seogeoaeo.ai/skills/seogeoaeo/SKILL.md here, update the README tool
list, and bump `version` in `server.json` and the plugin manifests.
