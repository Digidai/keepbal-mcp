# KeepBal MCP

Connect your AI client to the hosted KeepBal MCP service with one API key and one prepaid wallet.

**This repository contains connection documentation and configuration examples ONLY. The KeepBal gateway is NOT open-sourced.** The [MIT license](LICENSE) covers only the documentation and configurations in this repository; it does not license the hosted gateway or its source code.

| Setting | Value |
| --- | --- |
| Hosted endpoint | `https://keepbal.com/mcp` |
| Transport | Streamable HTTP |
| Authentication | `Authorization: Bearer $KEEPBAL_API_KEY` |
| Server name | `com.keepbal/keepbal` |
| Manifest | [server.json](server.json), mirrored from [the live manifest](https://keepbal.com/server.json) |

## Get a key

1. [Create a KeepBal account](https://keepbal.com/sign-up), or sign in to the [Console](https://keepbal.com/console).
2. In Console, open **API keys** and create a key.
3. Set `KEEPBAL_API_KEY` in the environment that launches Claude Code, Cursor, or Codex. The Desktop bridge has a separate local configuration step below.
4. Check your prepaid wallet balance in Console before running billed tools.

Send your key only to KeepBal. Keep real keys in your local environment or private client configuration, and never commit them. All examples here use placeholders or environment-variable references.

## Anonymous discovery and authenticated calls

| MCP method | API key required? |
| --- | --- |
| `initialize` | No |
| `tools/list` | No |
| `ping` | No |
| **Every `tools/call`** | **Yes, including free search and inspection** |

Connecting and seeing the tool list does not prove that your key is configured. Actual tool use requires the bearer header. The manifest's optional authorization header allows anonymous discovery; it does not make tool calls anonymous.

## Four tools and costs

| Tool | Main arguments | Purpose | Cost |
| --- | --- | --- | --- |
| `search_tools` | `query`; optional `category`, `limit`, `offset` | Find live tools and models. | Free; bearer key required. |
| `inspect_tool` | `tool` | Read a tool's known inputs, pricing metadata, and request example. | Free; bearer key required. |
| `run_tool` | `tool`, `input` | Execute a discovered tool using your prepaid wallet. | Billed according to the tool's pricing and execution outcome. |
| `get_run` | `run_id` | Check and settle an existing run. | No new execution or polling fee; the original run's charge may settle. |

Search, inspect, review the inputs and price, then run. Inspection returns pricing metadata, not an input-specific quote. Use `input.max_cost_usd` where a catalog tool needs a spending cap. Provider failures are not charged. Models appear in discovery but use their documented REST endpoints.

If `run_tool` reports a pending run, keep its `run_id` and call `get_run`. Do not repeat `run_tool` to check progress: that starts another execution. See the [MCP guide](https://keepbal.com/docs/mcp) and [billing documentation](https://keepbal.com/docs/billing) for current behavior.

## Client setup

Merge the relevant example into your existing client configuration, preserving other servers. Restart or reload the client after configuring it.

| Client | Example | Destination |
| --- | --- | --- |
| Claude Code | [claude-code.json](configs/claude-code.json) | `.mcp.json` in your project |
| Cursor | [cursor.json](configs/cursor.json) | `~/.cursor/mcp.json` or your project's `.cursor/mcp.json` |
| Claude Desktop | [claude-desktop.json](configs/claude-desktop.json) | `claude_desktop_config.json`, opened through **Settings → Developer → Edit Config** |
| Codex | [codex.toml](configs/codex.toml) | `~/.codex/config.toml` |

Claude Code expands `${KEEPBAL_API_KEY}` and Cursor expands `${env:KEEPBAL_API_KEY}` from their process environment. Make the variable available to the application before starting it; a GUI launch may not inherit your terminal's environment. References: [Claude Code MCP](https://code.claude.com/docs/en/mcp) and [Cursor MCP](https://cursor.com/docs/context/mcp).

**Claude Desktop bridge:** install Node.js with `npx` available to Desktop. The example runs [mcp-remote](https://github.com/punkpeye/mcp-remote) as a local stdio bridge to KeepBal's hosted Streamable HTTP endpoint. In your private Desktop configuration, replace the literal `$KEEPBAL_API_KEY` placeholder in `KEEPBAL_AUTH` with your key. Desktop's `env` values are literal strings; this placeholder is not shell-expanded. The bridge expands `${KEEPBAL_AUTH}` when building the header. Fully quit and reopen Desktop after saving.

**Codex:** `bearer_token_env_var = "KEEPBAL_API_KEY"` reads the key from the environment. Alternatively, register the same configuration with the CLI:

```sh
codex mcp add keepbal --url https://keepbal.com/mcp \
  --bearer-token-env-var KEEPBAL_API_KEY
```

See the [Codex MCP reference](https://developers.openai.com/codex/mcp).

## Install the KeepBal Skill

The canonical Skill is hosted at [https://keepbal.com/SKILL.md](https://keepbal.com/SKILL.md). Follow the [Skill installation example](configs/skill-install.md) for a local Claude Code or Codex installation. It downloads the hosted guide and adds skill metadata if needed. Configure MCP and your key separately using the examples above.

## Check the connection without a key

This initialization request needs no authorization header:

```sh
curl --fail-with-body --silent --show-error https://keepbal.com/mcp \
  --header 'Content-Type: application/json' \
  --header 'Accept: application/json, text/event-stream' \
  --data '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-06-18","capabilities":{},"clientInfo":{"name":"keepbal-connection-check","version":"1.0.0"}}}'
```

For a full MCP session, follow initialization with `notifications/initialized`, then `tools/list` or `ping`. Send one JSON-RPC message per POST and the negotiated `MCP-Protocol-Version` header on subsequent requests. KeepBal returns JSON responses and does not require a session ID.

Anonymous checks on 2026-10-28 confirmed successful initialization, tool listing, and ping, plus HTTP 401 for validly shaped calls to each of the four tools without a bearer key. Authenticated execution and billing were not exercised.

## Links

- [KeepBal documentation](https://keepbal.com/docs)
- [MCP connection guide](https://keepbal.com/docs/mcp)
- [Authentication](https://keepbal.com/docs/authentication)
- [Hosted Skill](https://keepbal.com/SKILL.md)
- [Tool catalog](https://keepbal.com/tools)
- [Console and API keys](https://keepbal.com/console)

`server.json` mirrors the hosted manifest using the `2025-12-11` schema, server name `com.keepbal/keepbal`, and remote type `streamable-http` at `https://keepbal.com/mcp`.
