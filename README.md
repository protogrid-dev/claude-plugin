# protogrid plugin for Claude

Find a Model Context Protocol (MCP) server for a task, compare the best candidates, and get a ready configuration for your MCP client.
Check your own remote MCP server before you submit it to the Claude or ChatGPT directories.

The plugin connects Claude to [protogrid](https://protogrid.dev), a public registry that mirrors the official MCP registry, probes every remote server without credentials, and scores trust and quality from what it can observe.
It is free, needs no account and no API key, and works in Claude (web, desktop, mobile, Cowork) and Claude Code.

## What it contains

- **MCP server** `protogrid` at `https://api.protogrid.dev/mcp` (streamable HTTP, no authentication), with five tools: `search`, `get_server`, `list_tools`, `get_connection` and `check_server`.
- **Skill `find-mcp-server`**: turns a task into a search, compares up to three candidates on how they connect, their quality and trust, and returns the configuration for the client you name (Claude Code, Codex, Cursor, VS Code, OpenCode, Gemini CLI, Goose or a generic `mcpServers` block).
- **Skill `check-mcp-server`**: checks a server URL once and explains the result: protocol support, tool definitions, authorization setup, a quality score, blockers and warnings for the Claude and ChatGPT directories, and the checklist of items nobody can see from outside.

No scripts, hooks or commands: the skills are instructions only, and nothing runs on your machine.

## Examples

- "Find an MCP server that can query a Postgres database and set it up for Claude Code."
- "Which MCP servers can read Jira issues without signing in?"
- "Check my MCP server at https://mcp.example.com/mcp before I submit it to the Claude directory."

## What it sends, and where

Everything goes to protogrid's MCP server at `api.protogrid.dev` and nowhere else:

- `search` sends the words of your query and any filters. protogrid keeps search text for 90 days, with a hashed client address that changes daily, to improve search.
- `get_server`, `list_tools` and `get_connection` send a server name.
- `check_server` sends the URL you ask it to check. protogrid then probes that URL once from its own servers, reads the protocol answer, the tool list and any public OAuth metadata, never sends credentials and never calls the server's tools. The result is kept for 30 days under a random link.

The skills never ask for API keys, tokens or passwords; configurations come back with placeholders where a secret goes, which you fill in on your own machine.
Scores come from observable signals; no audit is implied, and a passing check is not a directory approval.

Privacy policy: https://protogrid.dev/privacy.
Terms: https://protogrid.dev/terms.
Documentation: https://docs.protogrid.dev/reference/mcp-server/.

## Install

In Claude, add protogrid from the directory.
In Claude Code, this repository is also a plugin marketplace:

```bash
claude plugin marketplace add protogrid-dev/claude-plugin
claude plugin install protogrid@protogrid
```

## Support

Open an issue in this repository or write to hello@protogrid.dev.

## License

MIT.
