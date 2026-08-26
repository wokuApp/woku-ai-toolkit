# Changelog

## 0.2.0

Add two developer skills so the agent can build server-side woku integrations in code, not only operate the account through the MCP server:

- `woku-sdk-javascript`: use the official `@wokuapp/sdk` (Node.js) to manage trackers, VoC tools, sends, tickets and action plans.
- `woku-sdk-python`: use the official `woku` package (sync and async over httpx) with the same surface.

Both skills are auto-discovered from `skills/`; no manifest changes are needed to load them.

## 0.1.0

Initial release. Bundles woku's MCP server connection (`https://api.woku.app/mcp`) and the woku skill (the woku method plus the `woku_guide` / `design_voc_program` entry points). Plugin manifests for Claude Code (`.claude-plugin/`), Codex (`.codex-plugin/`), OpenClaw (`openclaw.plugin.json`) and Pi (`package.json`). OpenCode, claude.ai and ChatGPT connect to the MCP server through their own configuration.
