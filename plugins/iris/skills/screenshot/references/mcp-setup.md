# iris mcp — give the agent a `capture` tool

The same binary serves as a local **stdio MCP server** exposing one tool:
`capture`. It returns the image **inline** (base64 in the tool result) with
structured metadata, so the agent sees the pixels without locating a file.
Nothing is written to disk unless you pass `output`.

## Register the server

Claude Code / generic MCP client (`.mcp.json` at the repo root or settings):

```json
{
  "mcpServers": {
    "iris": {
      "command": "iris",
      "args": ["mcp"],
      "env": { "CHROME": "/path/to/chrome" }
    }
  }
}
```

Codex CLI:

```bash
codex mcp add iris -- iris mcp
```

If the Chrome binary is not auto-detected, pass `--chrome /path/to/chrome`
after `mcp`: `iris mcp --chrome /path/to/chrome` (see `iris mcp --help`).
Bare `localhost`, `.localhost`, and loopback addresses use HTTP automatically;
other bare hosts use HTTPS.

## The `capture` tool

Arguments (all optional except `url`):

```json
{
  "url": "http://localhost:3000",
  "selector": "#pricing-card",
  "padding": 24,
  "size": "desktop",
  "dark": false,
  "format": "png",
  "timeout_seconds": 30,
  "output": "/tmp/pricing-card.png"
}
```

`size`: `WxH` string or `desktop` | `iphone` | `ipad`. Omit `selector` for a
viewport/full capture; `padding` requires `selector`.

## MCP smoke test (CLI-side check, does not replace a real client call)

```bash
# server answers JSON-RPC on stdio; quick liveness check:
echo '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"t","version":"1"}}}' | iris mcp
# expect: {"jsonrpc":"2.0","id":1,"result":{"serverInfo":{"name":"iris","version":"..."}}}
```

## Notes

- MCP clients may need a **fresh agent task/session** before the newly
  registered `capture` tool appears.
- The MCP server keeps ONE Chrome process alive for its lifetime: first
  capture ~1 s, subsequent captures ~0.4 s (README reference: M2 Max).
- The CLI test does not prove the MCP path works — verify through the client.
