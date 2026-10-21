# TWOZERO MCP for Cursor

Adds the TWOZERO MCP server for TouchDesigner to Cursor.

When Cursor starts, the extension writes one entry to `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "twozero_td": {
      "url": "http://localhost:40404/mcp"
    }
  }
}
```

If `twozero_td` is already there, it changes nothing. To write the entry again, run **twozero MCP: Setup Connection** from the Command Palette.

## Before you start

Install the TWOZERO plugin in TouchDesigner: [download `twozero.tox`](https://twozero.ai/download/twozero.tox). The MCP server runs inside the plugin. Compatibility and setup for other agents: [README](https://github.com/twozero-team/twozero-td-mcp#readme).

## Install

1. Download `twozero-mcp-<version>.vsix` from [Releases](https://github.com/twozero-team/twozero-td-mcp/releases/latest).
2. In Cursor, open the Command Palette, run **Extensions: Install from VSIX...** and pick the file.
3. Reload Cursor.

## Build

```bash
npm ci
npx @vscode/vsce package
```

`vsce package` compiles the TypeScript first (`vscode:prepublish`). Release builds come from the [release workflow](https://github.com/twozero-team/twozero-td-mcp/blob/main/.github/workflows/release-extension.yml) when a `v*` tag is pushed.
