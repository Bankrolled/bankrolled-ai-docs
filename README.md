# Bankrolled.ai Agent Hub: docs

**Bankrolled.ai** is the agent hub for [Bankrolled](https://bankrolled.com). It serves **free**, agent-callable lookups of money facts and deposit-insurance schemes for English-speaking jurisdictions (US, UK, CA, AU, NZ). You can call it over plain REST or as a remote **MCP** server.

- **Canonical magazine: [bankrolled.com](https://bankrolled.com)**. The long-form guides live there. This hub does not republish article bodies.
- **Lookups are free.** No payment, API key or sign-up.
- **Sourced only.** Every answer carries the official regulator or government source and an `as_of` date. It also includes `source_url` (a bankrolled.com page) and a ready-made `cite` line. Unknown queries return `not_in_catalog`; numbers are never invented.
- Full agent docs: **https://bankrolled.ai/llms.txt**

This repository holds documentation only. The service runs at https://bankrolled.ai.

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
