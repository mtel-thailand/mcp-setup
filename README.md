# MTEL MCP setup

How to connect an AI agent (Claude Code, Cursor, VS Code, opencode, Claude
Desktop, …) to the MCP servers the MTEL team works with.

This repo holds **setup instructions only** — no code, no credentials. Every
server here authenticates with a token you create yourself; nothing in this
repo grants access to anything.

## Servers

| Server | What it gives your agent | Guide |
| ------ | ------------------------ | ----- |
| Plane  | Read and create work items on `task.mtel.cloud` | [servers/plane.md](servers/plane.md) |

## Adding a server to your client

Each guide gives a Claude Code command and the raw `command` / `args` / `env`
values. Any other client takes the same values in its own config shape.

Most clients take the `mcpServers` shape — Cursor (`.cursor/mcp.json` in a
project, or the global one), Windsurf, Claude Desktop, and others:

```json
{
  "mcpServers": {
    "<name>": {
      "command": "<command>",
      "args": ["<arg>", "<arg>"],
      "env": { "<KEY>": "<value>" }
    }
  }
}
```

Two known variations:

- **opencode** nests under `mcp`, marks the server `local`, takes the command
  as one array (check its docs for the name of the env block):

  ```json
  {
    "mcp": {
      "<name>": {
        "type": "local",
        "command": ["<command>", "<arg>", "<arg>"]
      }
    }
  }
  ```

- **VS Code** uses `.vscode/mcp.json` with a `servers` key rather than
  `mcpServers`, but the same `command` / `args` / `env` fields inside it.

Client config locations and key names move between versions — check your
client's own MCP documentation for where the file lives.

## Rules that apply to every server

- **Restart your session after adding a server.** MCP tools load once, at
  session start, in every client. A client can report a server as connected
  while the agent still cannot see its tools. In Claude Code, exit and run
  `claude --continue` to keep the conversation.
- **Run setup commands in your terminal, not in a chat with an agent.** The
  command text, token included, lands in the transcript.
- **Use your own token.** Each guide explains why for its server; the short
  version is attribution, per-person permissions, and revoking one token
  without breaking anyone else.

## Contributing a new server

Add `servers/<name>.md` following [servers/plane.md](servers/plane.md), and a
row to the table above. Every guide covers:

1. Prerequisites, with a command to check each one.
2. Where to create the token.
3. The Claude Code command, then the raw `command` / `args` / `env` values.
4. How to verify it works.
5. The mistakes that actually bite.

Never commit a real token, even an expired one. Use `<your token>`.
