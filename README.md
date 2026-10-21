# TWOZERO MCP for TouchDesigner

Connect Claude Code, Cursor or Codex to your live TouchDesigner session. The agent creates operators, wires networks, reads errors, runs Python and takes screenshots while you watch.

The MCP server runs inside the TWOZERO plugin, on your machine. The plugin and MCP are free.

This repository has the setup guide, the issue tracker and a Cursor extension that writes the MCP config for you.

[Download the plugin](https://twozero.ai/download/twozero.tox) · [Docs](https://twozero.ai/docs/mcp) · [Discord](https://discord.gg/zyTzVdAaSV)

## Compatibility

| What | Supported |
|---|---|
| TouchDesigner | Free, Commercial or Pro, recent build. The minimum build moves with each plugin release: see [System Requirements](https://twozero.ai/docs/requirements). The installer checks your build and offers a download link if it is too old. |
| Windows | 10 or newer |
| macOS | Apple silicon (M1 or newer). TouchDesigner needs Full Disk Access to install the plugin. |
| Agents | Claude Code (in the terminal or in the Code section of Claude Desktop), Cursor, Codex. Nothing extra to install. |
| Claude Desktop, Chat section | Connects through `npx mcp-remote` and needs Node.js. Use the Code section instead. |

Use the strongest model your agent offers, such as Claude Opus. Smaller models get operator parameters wrong more often.

## Install

1. Download [`twozero.tox`](https://twozero.ai/download/twozero.tox).
2. Close your project and double-click `twozero.tox`. TouchDesigner opens it as a project of its own.
3. Click **GET STARTED**.
   - Windows: confirm the permission prompt.
   - macOS: if the installer stops on **One more step**, click **OPEN SETTINGS & QUIT TD**. Turn on Full Disk Access for TouchDesigner, then open `twozero.tox` again.
4. The agent connection wizard opens. Click **CONNECT** next to your agent.
5. Reload the agent: run `/mcp` in Claude Code, reload the Cursor window, restart `codex`, or quit and reopen Claude Desktop.
6. Open your project. Then open a new chat and ask:

   ```
   Show me what you can see in my TD project.
   ```

The plugin installs once per TouchDesigner build and loads with every project. The MCP server starts with TouchDesigner: **Settings > MCP > Auto start** is on by default.

An agent connects once, when the chat starts, and does not retry. Start TouchDesigner first, then open a new chat.

The MCP status in the TWOZERO window reads `connected` when the server is up. If MCP is off, turn it on:

![Turn on MCP](assets/mcpOn.gif)

To connect more agents later, open **Settings > Reconnect agents**. Video and details: [Installation](https://twozero.ai/docs/installation).

## Manual setup

The wizard adds one server named `twozero_td` with the URL `http://localhost:40404/mcp`. To add it by hand:

### Claude Code

```bash
claude mcp add --transport http --scope user twozero_td http://localhost:40404/mcp
```

Check with `claude mcp list`.

### Cursor

Add to `~/.cursor/mcp.json`, then reload the Cursor window:

```json
{
  "mcpServers": {
    "twozero_td": {
      "url": "http://localhost:40404/mcp"
    }
  }
}
```

### Codex

Add to `~/.codex/config.toml`, then restart Codex:

```toml
[mcp_servers.twozero_td]
url = "http://127.0.0.1:40404/mcp"
```

### Ask your agent

```
Add twozero TD MCP for me with server key "twozero_td" and URL "http://localhost:40404/mcp". Configure it globally (user scope) so it is available from any project/workspace.
```

Add one entry, even with several TouchDesigner instances open. Each instance takes the next free port (40404, 40405 and so on), and the agent asks which one to work in. The base port is in **Settings > MCP > Port**.

## Cursor extension

The [extension](extension/) adds the `twozero_td` entry to `~/.cursor/mcp.json` when Cursor starts. If the entry is already there, it changes nothing. It writes Cursor's config only.

1. Download `twozero-mcp-<version>.vsix` from [Releases](https://github.com/twozero-team/twozero-td-mcp/releases/latest).
2. In Cursor, open the Command Palette, run **Extensions: Install from VSIX...** and pick the file.
3. Reload Cursor.

To write the entry again, run **twozero MCP: Setup Connection** from the Command Palette.

## Talking to the agent

Start each new chat with a prompt that sends the agent into TouchDesigner:

```
Show me what you can see in my TD project.
check in TD: <your task>
*here in TD: <your task>
Check *this op in TD: <your task>
```

Once it confirms what it sees, talk normally.

### Pointers

| Write | The agent works on |
|---|---|
| `*here`, `*тут` | the network you are looking at |
| `*this`, `*этот`, `*эту` | the selected operator |
| `*these`, `*эти` | all selected operators |

The asterisk never shows up in normal code or text, so the agent reads it as an exact pointer. Plain "here" and "this op" work too.

To hand over exact operators, select them in the network editor and press Ctrl+C (Cmd+C on macOS). TWOZERO copies their paths. Paste them into the chat and write the task around them:

```
/project1/base1/noise1, /project1/base1/blur1 - make these two audio-reactive
```

### Task prompts

| Task | Prompt |
|---|---|
| Errors | `Check errors across the whole project and show what is critical first.` |
| Performance | `Analyze what is causing lag and show the top slow operators.` |
| Compare | `Compare container A and B: structure, parameters, and connections; list the differences.` |
| Selected operator | `Inspect this selected operator and nearby chain; do not change anything until I confirm.` |
| Plan first | `Diagnostics and plan first, changes only after approval.` |
| Verify | `After applying fixes, run error and performance checks again to verify.` |

### Built-in triggers

| Phrase | What happens |
|---|---|
| `study this project`, `изучи проект` | A tour of an unfamiliar or large project. |
| `twozero bug report`, `twozero баг репорт` | The built-in bug report flow. The word `twozero` is required, so a plain "report a bug" does not start it. |

### Russian

Mix English and Russian freely. TD terms (`op`, `par`, `comp`) work without translation.

| English | Russian |
|---|---|
| TouchDesigner, TD | тач |
| project, patch, .toe file | проект, патч |
| operator, op | оператор, оп |
| parameter, par | параметр, пар |
| `*here in TD` | `в таче *тут` |
| `*this op in TD` | `в таче *этот оп` |
| `*these ops in TD` | `в таче *эти опы` |
| `Show me what you can see in my TD project.` | `Посмотри, что у меня в таче.` |

## Troubleshooting

| Problem | What to do |
|---|---|
| The agent doesn't see the `twozero_td` tools | The chat started before TouchDesigner finished loading. With TD up, open a new chat. Reconnecting inside the old chat does not help. |
| Still nothing in a new chat | Run `curl http://localhost:40404/`. JSON back means the server runs. *Connection refused* means it never started: check that TWOZERO runs in TD and the port is free, and on macOS check the firewall. In Claude Code, try `/mcp` > `twozero_td` > reinitialize. |
| Not sure the config is right | **Settings > Reconnect agents** shows which agents are connected and rewrites the entry. |
| Commands fail silently | Your TouchDesigner build is older than the plugin needs. See [System Requirements](https://twozero.ai/docs/requirements). |

Keep **Settings > MCP > Local only** set to Yes, so only your machine can reach the server.

More fixes: [Troubleshooting](https://twozero.ai/docs/troubleshooting).

## Support

- Questions and feedback: [Discord](https://discord.gg/zyTzVdAaSV).
- Bugs: [open an issue](https://github.com/twozero-team/twozero-td-mcp/issues/new/choose) with your TouchDesigner build, OS and the plugin version from Settings. Or type `twozero bug report` in the agent chat.
- Security issues: report them privately, see the [security policy](https://github.com/twozero-team/twozero-td-mcp/security/policy).
- Contributing: see [CONTRIBUTING.md](https://github.com/twozero-team/.github/blob/main/CONTRIBUTING.md).
