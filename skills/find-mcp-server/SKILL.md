---
name: find-mcp-server
description: Find a Model Context Protocol (MCP) server for a task the user describes, compare the best candidates on how they connect, their quality and trust, and give back a ready configuration for the user's MCP client. Use when the user asks for an MCP server, connector or integration that does something (for example "an MCP server to query Postgres" or "how can my agent read Jira issues"), or wants to install or configure one. Not for testing a server the user runs themselves; use check-mcp-server for that.
---

# Find an MCP server and connect it

protogrid indexes every server of the official MCP registry and probes the remote ones without credentials.
Its tools return what each server offers and exactly how to connect to it.

## Steps

1. Turn the request into a short plain-language query about the task, without client or vendor names unless the user asked for a specific product.
   For example "query a postgres database" or "create jira issues".
2. Call `search` with `q`, `limit: 10` and `exclude_quality_flags: ["known-vulns"]`.
   - If the user needs something that works with no sign-in, add `class: ["R0"]`; with an API key they hold, `class: ["R0", "R1"]`.
   - If the user asked only for servers whose owner is known, add `owner_verified: true`.
   - If nothing comes back, try once more with broader words and without `exclude_quality_flags`, and say so.
3. Pick at most three candidates.
   Prefer results whose `quality_label` is `strong` or `good`, whose `matched_tools` fit the task, and with fewer `trust_flags`.
   Call `get_server` with `name` for each to read its tools, its connection class and the health of its remotes.
4. Present two or three candidates when the search found that many, so the user can choose, one short paragraph each:
   - what it does, from its tools;
   - how it connects, in plain words:
     - `R0`: works right away, no sign-in;
     - `R1`: needs an API key or token that the user creates with that provider (say which placeholder in the returned block stands for it);
     - `R2`: the user signs in once in a browser (OAuth);
     - `L0`: runs a package on the user's own machine; always say plainly that this means running third-party code there, with the user's permissions;
   - its quality label and score, its trust score, and any flags, in plain words;
   - "owner verified" only means the publisher proved control of the name; it is not an audit.
5. Ask which client the user wants to set it up in, unless they already said.
   Then call `get_connection` with the chosen `name` and the matching `target`:
   Claude Code `claude-code-cli`, Codex `codex-toml`, Cursor `cursor`, VS Code `vscode`, OpenCode `opencode`, Gemini CLI `gemini`, Goose `goose`, anything else `mcpServers`.
6. Show the returned configuration exactly as returned, in a code block, and say where it goes for that client.
   Secret values in the block are placeholders (a variable name between braces after a dollar sign): the user replaces them on their own machine.
   If the server needs a setting the block does not carry (for example a database connection string that only its documentation mentions), say so after the block and show your addition separately, marked as coming from the server's documentation rather than from the registry; never change the returned block silently.
7. End with the server's page, `https://protogrid.dev/servers/<name>`, where the full quality and trust details are.

## Rules

- Never ask the user to paste an API key, token or password into the conversation, and never fill a placeholder yourself.
  If the user pastes a secret anyway, tell them to keep it out of the chat and do not repeat it.
- Never describe a server as safe, secure, audited or approved.
  Scores come from observable signals such as protocol support, tool definitions, dependencies and uptime; no code audit is implied.
- Recommend only servers that the tools returned; do not invent names, tools or configuration fields.
- You cannot call the found server's tools from here; the user connects it in their own client.
- Some categories are not listed at all for anonymous use; if nothing fits, say that nothing suitable was found rather than guessing.
