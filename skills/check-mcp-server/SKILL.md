---
name: check-mcp-server
description: Check a remote MCP server from its public URL, without credentials and without calling its tools, and explain the result - protocol support, tool definitions, authorization setup, a quality score, and readiness for the ChatGPT (OpenAI) and Claude directories. Use when the user wants to test, validate or debug an MCP server they build or run, asks why it could fail a directory review, or prepares a directory submission.
---

# Check an MCP server

`check_server` probes a public MCP server URL once.
It never sends credentials and never calls the server's tools; it reads the protocol answer, the tool list and any OAuth metadata.

## Steps

1. Get the server's public URL: its streamable HTTP endpoint (often ending in `/mcp`) or its SSE endpoint.
   - If the URL contains a password, token, key or similar value (in the user part or the query), do not use it.
     Ask for the URL without it; checks never take credentials.
   - Local or private addresses (localhost, 127.0.0.1, 10.x, 192.168.x) cannot be checked; the server must be reachable from the internet.
2. Call `check_server` with `url`.
   If the answer has `status` `queued` or `running`, call `check_server` with the returned `id` until it is `done` or `failed`, at most three times.
   `failed` means the check itself could not be completed (not that the server failed); suggest trying again later.
3. If the call returns an error, explain it in plain words:
   - `invalid_url`: repeat the message, which says what is wrong with the URL;
   - `check_quota_exceeded`: say when another check is possible (`retry_after_seconds`);
   - anything else: say the check could not run and suggest trying again later.
4. Read `result.probe.outcome` first.
   - `ok`: the server answered and listed its tools.
   - `auth_required`: the server asks for sign-in (OAuth or a key); the authorization checks still apply.
   - Anything else (could not connect, timed out, not an MCP server, blocked): explain `result.probe.error` and the usual causes, such as a wrong path, a server that is down, a firewall, or a URL that serves a web page instead of MCP.
     Stop there; nothing else can be judged.
5. Report the quality: `result.quality.score` and `result.quality.label`, then every check in `result.quality.checks` whose `status` is `fail` or `warn`, grouped by `category`, each with its `detail` and a concrete fix.
   A one-off check has no history, so its label is never `strong` or `new`.
6. Report directory readiness, one directory at a time from `result.readiness`, in this order:
   - blockers: items with `level` `must` and `status` `fail`; these would stop a submission, so explain each `detail` and how to fix it;
   - warnings: other `fail` and `warn` items of kind `auto`;
   - review: `warn` items of kind `heuristic`; these are guesses about wording (for example tool descriptions that steer the model); ask the user to judge them;
   - unknown: what the probe could not observe;
   - manual: requirements nobody can see from outside (privacy policy, support contact, test account, screenshots); give them as a short checklist.
7. Give the shareable result link from `page`.
8. Suggest next steps that fit:
   - fix and check again (the same URL within five minutes returns the recent result);
   - run the same check in CI after every deploy with the GitHub Action `protogrid-dev/check-action` (it needs a free API key from protogrid.dev);
   - publish the server to the official MCP registry and claim it on protogrid.dev for monitoring and alerts.

## Rules

- A passing check is not an approval: both directories also review submissions by hand.
- Never describe the server as safe, secure or audited; the check only reads what the server exposes publicly.
- Check only the URLs the user gives, a few at a time; checks are rate limited per hour.
