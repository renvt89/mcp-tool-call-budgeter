# MCP Tool Call Budgeter

A small, zero-dependency Node (>=18) **stdio proxy** that sits between an MCP client and an MCP server. It caps `tools/call` requests per session and per tool, and writes a JSON overage report when the server exits.

> This repo is **documentation only**. The script (`budgeter.mjs`, about 4 KB) and an example budget file are sold on Gumroad: **https://renevibe76.gumroad.com/l/xzwtga**

## What it does
- Counts every `tools/call` JSON-RPC request by tool name.
- Blocks a call with a JSON-RPC error (`code -32000`, `budget exceeded: ...`) when any of these is true:
  - the session cap (`sessionMaxCalls`) is reached,
  - the tool's own cap (`tools.<name>.maxCalls`, otherwise `defaultMaxCalls`) is reached,
  - the call arguments are larger than `maxArgBytes`.
- Forwards every other message unchanged.
- On exit it writes a report with total calls, per-tool calls, caps and blocked counts, a list of the blocked calls, and an `overage` flag.

## What it does NOT do (honest limits)
- It does **not** count LLM tokens. It caps tool *calls* and argument *bytes*. Token caps need your provider's usage data.
- It supports **stdio only**. There is no HTTP or SSE transport.
- It is **not a security sandbox**. A cap of 0 blocks calls through this proxy only.
- It was not run end-to-end in our build environment. Test it with your own server before you rely on it.

## Usage (after purchase)
```
node budgeter.mjs --budget budget.json --report budget-report.json -- node your-mcp-server.js
```
In your MCP client config, set the server command to `node` with the args `/path/budgeter.mjs --budget /path/budget.json -- <your real server command>`.

## Example budget file shape
```json
{
  "sessionMaxCalls": 200,
  "defaultMaxCalls": 50,
  "maxArgBytes": 20000,
  "tools": { "web_fetch": { "maxCalls": 20 } }
}
```
All numbers are placeholders. Tune them for your own server.

## Related
- MCP Manifest Breaking-Change Diff: https://github.com/renvt89/mcp-manifest-diff

## Disclosure
Built with AI assistance by Rene Vergara. MIT license for the purchased script.
