<p align="center">
  <img src="assets/logo-512.png" alt="seogeoaeo.ai" width="96" height="96">
</p>

# seogeoaeo.ai for AI agents

Check any public website for SEO, AEO and GEO readiness from your coding agent.
This repo packages the [seogeoaeo.ai](https://seogeoaeo.ai) MCP server, REST API
and agent skill so Claude Code, Cursor, Codex, VS Code and any other MCP client
can install them in one step.

Each tool is a single check. A run returns findings with a severity, why each
one matters and how to fix it, plus a prioritized list of changes your agent can
apply. Re-run the same tool afterwards to verify the fix.

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
`curl`, check the credit balance first, retry safely and apply the returned fixes.
The always-current copy is served at
[`seogeoaeo.ai/skills/seogeoaeo/SKILL.md`](https://seogeoaeo.ai/skills/seogeoaeo/SKILL.md).

## What the agent can do

Start with one website assessment (`run-assessment`). It runs the core checks,
scores SEO readiness, AEO readiness and AI access from 0 to 100 like PageSpeed
Insights, and returns one fix list ordered by priority. Run it again after
deploying to see the percent of earlier issues fixed.

For a focused question, run a single tool. Every tool is its own MCP tool, named
by its slug:

**SEO**

- `indexing-checker`: Indexing & Canonical Checker
- `robots-checker`: Robots.txt & AI Crawler Checker
- `redirects-headers-checker`: Redirects, Headers & Host Checker
- `on-page-seo-checker`: On-Page SEO Checker
- `xml-sitemap-validator`: XML Sitemap Validator
- `http-status-bulk-checker`: HTTP Status Bulk Checker
- `broken-internal-link-finder`: Broken Internal Link Finder
- `open-graph-checker`: Open Graph & Social Card Checker
- `readability-analyzer`: Readability Analyzer
- `keyword-ideas`: Keyword Ideas
- `keyword-clusterer`: Keyword Clusterer
- `keyword-deduplicator`: Keyword Deduplicator
- `core-web-vitals-snapshot`: Core Web Vitals Snapshot
- `page-size-checker`: Page Size & 2 MB Limit Checker
- `hreflang-checker`: Hreflang Checker
- `favicon-checker`: Favicon & Site Name Checker
- `indexnow-checker`: IndexNow Checker & Submitter
- `mobile-parity-checker`: Mobile Parity Checker
- `product-schema-checker`: Product Schema Checker
- `local-business-checker`: Local Business Checker
- `image-seo-checker`: Image SEO Checker
- `orphan-page-finder`: Orphan Page Finder

**AEO**

- `schema-checker`: Schema Markup Checker
- `schema-markup-generator`: Schema Markup Generator
- `organization-entity-markup-builder`: Organization Entity Markup Builder
- `faq-schema-builder`: FAQ Schema Builder
- `breadcrumb-schema-builder`: Breadcrumb Schema Builder
- `content-freshness-checker`: Content Freshness Checker
- `author-eeat-checker`: Author & E-E-A-T Checker
- `question-finder`: Question Finder

**GEO**

- `ai-agent-readiness`: AI Agent Readiness Checker
- `llms-txt-generator`: llms.txt Generator
- `citability-analyzer`: Citability Analyzer
- `ai-bot-log-analyzer`: AI Bot Log Analyzer
- `ai-crawler-view`: AI Crawler View
- `brand-entity-checker`: Brand Entity Checker
- `fanout-coverage-checker`: Fan-out Coverage Checker
- `ai-answer-visibility`: AI Answer Visibility
- `brand-narrative-check`: Brand Narrative Check

Helper tools: `check-credits`, `get-run`, `get-assessment`.

## Pricing

Each tool is priced in credits, and the price is in the tool's description. Most
checks cost 3 to 20 credits. The three tools that sample AI answers cost more
(`fanout-coverage-checker` 95, `ai-answer-visibility` 120,
`brand-narrative-check` 150) because they call paid model providers, and the
agent is told to run them only when you ask. A one-page website assessment costs
67 credits. Failed runs are refunded.

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

This repo holds configuration only. The service itself is hosted at
[seogeoaeo.ai](https://seogeoaeo.ai).

## Support

Questions or problems: [seogeoaeo.ai/contact](https://seogeoaeo.ai/contact).

## License

[MIT](LICENSE) for the contents of this repo. Use of the hosted service is
covered by the [terms](https://seogeoaeo.ai/terms) and
[privacy policy](https://seogeoaeo.ai/privacy).
