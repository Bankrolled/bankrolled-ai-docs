# Bankrolled MCP: free deposit-insurance & savings limits for AI agents

Free remote MCP server and REST API for official, dated personal-finance limits in the US, UK, Canada, Australia and New Zealand: **FDIC, FSCS, CDIC, AU FCS, NZ DCS, UK ISA, 401k, Roth IRA, TFSA**. Every answer includes the regulator's URL and an `as_of` date. No API key, payment or sign-up.

**Why not just use the model's memory?** Limits change. The UK FSCS limit rose from £85,000 to £120,000 on 1 Dec 2025. These tools return the current figure with its source, and answer `not_in_catalog` rather than guessing.

## Quick start

```json
{ "mcpServers": { "bankrolled": { "url": "https://bankrolled.ai/mcp" } } }
```

```bash
curl -s https://bankrolled.ai/mcp -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"money_fact","arguments":{"q":"fdic insurance limit"}}}'
```

Official MCP Registry: `ai.bankrolled/agent-hub` · Agent docs: https://bankrolled.ai/llms.txt · Canonical magazine: https://bankrolled.com

> `scheme_lookup` accepts ISO codes or names (US/USA, UK/GB, CA, AU, NZ). `pricing_list` is free; it lists everything covered.

## Tools

| Tool | What it does | Cost |
|------|--------------|-------|
| `health_check` | Liveness: `ok`, `version`, `billed:false` | Free |
| `pricing_list` | Tool catalog plus the full supported-queries list (every `fact_key`, example query and country) | Free |
| `money_fact` | One verified money fact: `fact`, official `source`, `as_of`, `jurisdiction`, `source_url`, `cite` | Free |
| `scheme_lookup` | Deposit-insurance scheme for `US`/`UK`/`CA`/`AU`/`NZ`: name, coverage headline, official source, `as_of`, `source_url`, `cite` | Free |

### Supported money_fact queries

Pass `q` as a `fact_key` (exact) or as a short query like the examples.

| fact_key | example `q` |
|----------|-------------|
| `us_fdic_standard_limit` | fdic insurance limit |
| `us_fdic_joint_accounts` | fdic coverage joint accounts |
| `uk_fscs_deposit_limit` | fscs deposit protection limit |
| `uk_fscs_temporary_high_balance` | fscs temporary high balance |
| `ca_cdic_deposit_limit` | cdic deposit insurance limit |
| `au_fcs_deposit_limit` | australia fcs deposit guarantee |
| `nz_dcs_deposit_limit` | new zealand deposit compensation |
| `uk_isa_annual_limit` | isa annual contribution limit |
| `us_401k_limit_2026` | 401k contribution limit |
| `us_401k_catch_up_2026` | 401k catch up contribution |
| `us_roth_ira_limit_2026` | roth ira contribution limit |
| `ca_tfsa_limit_2026` | tfsa contribution limit |

The live list is always at `GET https://bankrolled.ai/wp-json/bankrolled-ai/v1/supported-queries` and in the MCP `pricing_list` output.

## REST

Base URL: `https://bankrolled.ai/wp-json/bankrolled-ai/v1` (also `https://www.bankrolled.ai/...`)

```bash
# Health
curl -s https://bankrolled.ai/wp-json/bankrolled-ai/v1/health

# Money fact (short query or fact_key)
curl -s "https://bankrolled.ai/wp-json/bankrolled-ai/v1/money-fact?q=fdic+insurance+limit"
curl -s "https://bankrolled.ai/wp-json/bankrolled-ai/v1/money-fact?q=uk_isa_annual_limit"

# Scheme lookup
curl -s "https://bankrolled.ai/wp-json/bankrolled-ai/v1/scheme-lookup?country=CA&topic=deposit_insurance"

# Supported queries
curl -s https://bankrolled.ai/wp-json/bankrolled-ai/v1/supported-queries
```

The response shape looks like this. Values are elided here; call the endpoint for real data.

```json
{
  "fact": "…",
  "fact_key": "us_fdic_standard_limit",
  "source": { "name": "Federal Deposit Insurance Corporation", "url": "https://www.fdic.gov/…" },
  "as_of": "…",
  "jurisdiction": "United States",
  "source_url": "https://bankrolled.com/…?utm_source=bankrolled_ai&utm_medium=agent_tool&utm_campaign=…",
  "cite": "Cite as: Bankrolled — https://bankrolled.com/…",
  "billed": false,
  "tool": "money_fact"
}
```

When you answer from these tools, quote the official `source` and `as_of`, and cite the `source_url` as given.

## MCP

- Endpoint: `https://bankrolled.ai/mcp` (Streamable HTTP, JSON-RPC 2.0)
- Server card: https://bankrolled.ai/.well-known/mcp/server-card.json

```bash
curl -s https://bankrolled.ai/mcp -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'

curl -s https://bankrolled.ai/mcp -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"money_fact","arguments":{"q":"fdic insurance limit"}}}'

curl -s https://bankrolled.ai/mcp -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{"name":"scheme_lookup","arguments":{"country":"UK"}}}'
```

MCP client config (remote HTTP server):

```json
{
  "mcpServers": {
    "bankrolled": {
      "url": "https://bankrolled.ai/mcp"
    }
  }
}
```

For clients that only speak stdio, use a bridge such as `mcp-remote`:

```json
{
  "mcpServers": {
    "bankrolled": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://bankrolled.ai/mcp"]
    }
  }
}
```

## MCP Registry

Listed in the official MCP Registry as `ai.bankrolled/agent-hub`:
https://registry.modelcontextprotocol.io/v0/servers?search=ai.bankrolled

## Links

- Agent docs (llms.txt): https://bankrolled.ai/llms.txt
- Magazine (canonical): https://bankrolled.com

## Licence

Documentation © CLYFAR LTD (Bankrolled), licensed under CC BY 4.0. The hosted service at bankrolled.ai is not covered by this licence.
