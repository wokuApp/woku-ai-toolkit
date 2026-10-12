# Changelog

## 0.3.0

Also the per-tool metrics of a folder (`get_folder_tools_metrics`, `folders.metrics`), tracker chips on `search_voc_tools` and `tools.list` (`trackers`) and `integrationProvider` on integration folders.

Cover folders and the unified tools view: the woku skill documents the MCP tools
`search_voc_tools`, `get_voc_tools_stats`, `move_voc_tools`, `update_folder` (also sets the folder's `whatsappAlertRecipients`),
`delete_folder`, `get_folder_report` (`scope: "all"`, `format: "csv"`), `folderId`
on `create_nps_tool`, `create_csat_tool` and `create_ces_tool`, and the
`organize_listening` playbook, flows with ordered steps (`create_flow`, `update_flow`, `get_flow` with `steps`), and journeys in folders (`get_journey_folders_status`, `organize_journey_folders`, `folderId` and `momentFolders` on journeys). The JavaScript and Python SDK skills add the
`folders` and `tools` resources. Requires the server and SDK releases that ship them. The skills also cover the folder PDF in the SDKs (`folders.pdf`) and the folders paging (`foldersLimit`, `foldersCursor`) of `search_voc_tools` and `tools.list`.

## 0.2.2

Reconcile journey/MCP workflows, reviewed proposals, local media, SDK v4 methods,
retry boundaries and instrument selection in the shipped skills. Correct NPS
variable examples and public cursor limits.

## 0.2.1

Remove the action-plan external-send example (Jira/Monday/ClickUp/Notion) from the SDK skills: those destinations are not available in production and the SDKs dropped `actionPlans.send`. The skills now show approve plus manage-in-woku.

## 0.2.0

Add two developer skills so the agent can build server-side woku integrations in code, not only operate the account through the MCP server:

- `woku-sdk-javascript`: use the official `@wokuapp/sdk` (Node.js) to manage trackers, VoC tools, sends, tickets and action plans.
- `woku-sdk-python`: use the official `woku` package (sync and async over httpx) with the same surface.

Both skills are auto-discovered from `skills/`; no manifest changes are needed to load them.

## 0.1.0

Initial release. Bundles woku's MCP server connection (`https://api.woku.app/mcp`) and the woku skill (the woku method plus the `woku_guide` / `design_voc_program` entry points). Plugin manifests for Claude Code (`.claude-plugin/`), Codex (`.codex-plugin/`), OpenClaw (`openclaw.plugin.json`) and Pi (`package.json`). OpenCode, claude.ai and ChatGPT connect to the MCP server through their own configuration.
