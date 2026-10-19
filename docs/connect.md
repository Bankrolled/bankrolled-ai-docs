# Connect Bankrolled to your AI client

Bankrolled is a free remote MCP server with **Streamable HTTP** transport at:

```
https://bankrolled.ai/mcp
```

It has no API key, no OAuth, no sign-up and no payment. All four tools are read-only and free: `money_fact`, `scheme_lookup`, `pricing_list` and `health_check`.

Every config below was checked against the live server on 19 Oct 2026. See "Tested" under each one.

---

## Claude Code (CLI)

```bash
claude mcp add --transport http bankrolled https://bankrolled.ai/mcp
claude mcp list        # should show: bankrolled: https://bankrolled.ai/mcp (HTTP) - ✔ Connected
```

You can also add it as JSON (for example to share with a team via `.mcp.json` at the repo root):

```bash
claude mcp add-json bankrolled '{"type":"http","url":"https://bankrolled.ai/mcp"}'
```

```json
{
  "mcpServers": {
    "bankrolled": { "type": "http", "url": "https://bankrolled.ai/mcp" }
  }
}
```

*Tested:* Claude Code 2.1.303. `claude mcp add` then `claude mcp list` and `claude mcp get` both report ✔ Connected.

---

## Claude Desktop / claude.ai

**Option A: custom connector (no config file).** Go to Settings → Connectors → *Add custom connector*. Use the name `Bankrolled` and the URL `https://bankrolled.ai/mcp`. Leave the OAuth fields empty. Custom connectors depend on your Claude plan.

**Option B: `claude_desktop_config.json` through the `mcp-remote` bridge.** This needs Node.js 18 or later.
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`

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

Restart Claude Desktop after saving.

*Tested:* `mcp-remote` 0.15.1 with this exact command. It reported "Connected to remote server using StreamableHTTPClientTransport". `initialize`, `tools/list` and `tools/call` (money_fact) all returned results over stdio. The connector UI (option A) can't be driven from a server, so it wasn't clicked through.

---

## Cursor

Use `~/.cursor/mcp.json` (global) or `.cursor/mcp.json` (per project):

```json
{
  "mcpServers": {
    "bankrolled": {
      "url": "https://bankrolled.ai/mcp"
    }
  }
}
```

Then open Settings → MCP and check that **bankrolled** shows 4 tools.

*Tested:* the JSON is valid and in Cursor's documented remote-server shape. The transport it relies on (Streamable HTTP, JSON responses) was exercised with the official MCP TypeScript SDK. Cursor itself wasn't run.

---

## VS Code (GitHub Copilot agent mode)

Use `.vscode/mcp.json` in your workspace, or run **MCP: Open User Configuration** for your profile:

```json
{
  "servers": {
    "bankrolled": {
      "type": "http",
      "url": "https://bankrolled.ai/mcp"
    }
  }
}
```

There is also a portable format, `.mcp.json` at the workspace root, which Copilot sessions read too:

```json
{
  "mcpServers": {
    "bankrolled": { "type": "http", "url": "https://bankrolled.ai/mcp" }
  }
}
```

*Tested:* the JSON is valid and matches the VS Code MCP configuration reference (`type: "http"` plus `url`; VS Code tries Streamable HTTP first). The transport was tested with the official SDKs. VS Code itself wasn't run.

---

## ChatGPT (custom MCP server / plugin)

1. On chatgpt.com, open **Plugins** and select **+**, then **Add custom MCP server**.
2. Name: `Bankrolled`. Server URL: `https://bankrolled.ai/mcp`.
3. Authentication: **No authentication**.
4. Tick *I understand and want to continue*, then select **Create as a plugin**. ChatGPT lists the 4 tools it discovered.
5. Install the plugin. In a new chat, type `@` and pick **Bankrolled**.

Which accounts can add custom MCP servers depends on your plan and workspace settings. On Business or Enterprise workspaces, an admin may need to allow it.

*Tested:* the server's no-auth behaviour was checked. `initialize` and `tools/list` need no credentials. The OAuth discovery URLs (`/.well-known/oauth-protected-resource`, `/.well-known/oauth-authorization-server`) return 404, which is what a "No authentication" server should do. The ChatGPT UI itself wasn't clicked through.

---

## Any Streamable HTTP client

**Raw JSON-RPC (curl):**

```bash
curl -s https://bankrolled.ai/mcp \
  -H 'content-type: application/json' \
  -H 'accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"money_fact","arguments":{"q":"fscs deposit protection limit"}}}'
```

**TypeScript, official SDK** (`npm i @modelcontextprotocol/sdk`):

```js
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { StreamableHTTPClientTransport } from '@modelcontextprotocol/sdk/client/streamableHttp.js';

const client = new Client({ name: 'my-agent', version: '1.0.0' });
await client.connect(new StreamableHTTPClientTransport(new URL('https://bankrolled.ai/mcp')));
const res = await client.callTool({ name: 'money_fact', arguments: { q: 'fscs deposit protection limit' } });
console.log(JSON.parse(res.content[0].text));   // fact, source {name,url}, as_of, jurisdiction, cite
await client.close();
```

**Python, official SDK** (`pip install mcp`, tested with 2.3.0):

```python
import asyncio, json, httpx2
from mcp import ClientSession
from mcp.client.streamable_http import streamable_http_client

async def main():
    async with httpx2.AsyncClient() as http, \
               streamable_http_client("https://bankrolled.ai/mcp", http_client=http) as streams:
        async with ClientSession(streams[0], streams[1]) as s:
            await s.initialize()
            res = await s.call_tool("money_fact", {"q": "hsa contribution limit"})
            print(json.loads(res.content[0].text))

asyncio.run(main())
```

On mcp 1.x the helper is called `streamablehttp_client(url)` and yields `(read, write, get_session_id)`.

*Tested:* TS SDK 1.32.1 negotiated protocol 2025-11-25 and ran `tools/list`, `prompts/list` and `money_fact`. Python SDK 2.3.0 ran `initialize`, `tools/list` and `money_fact`.

---

## What to expect

- **Tools:** `money_fact(q)`, `scheme_lookup(country)`, `pricing_list()`, `health_check()`.
- **Fields on every answer:** `fact`, `source.name`, `source.url` (the official regulator or tax-authority page), `as_of`, `jurisdiction`, `cite` and `billed: false`.
- **Unknown questions** return `error: "not_in_catalog"` with the list of supported `fact_key`s. Nothing is guessed.
- **Coverage:** 24 facts across the US, UK, Canada, Australia and New Zealand. Call `pricing_list` for the live list.
- **Fair use:** bursts are rate-limited per IP. On HTTP 429, wait 10 seconds.
