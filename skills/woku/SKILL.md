---
name: woku
description: "Use woku to protect revenue with the voice of the customer. Use when the user wants to set up customer feedback or listening, design a Voice-of-Customer program, evaluate courses, teachers, trainings, events or teams organized in folders, place surveys (NPS, CSAT, CES) or wokus at the moments of a customer journey, turn feedback into support tickets or action plans, or operate their woku account from the agent. Requires the woku MCP server connected (this toolkit declares it)."
---

# woku: voice-of-customer for your agent

woku is a voice-of-customer platform. This toolkit connects your agent to woku's MCP server (`https://api.woku.app/mcp`), which exposes the full tool catalog plus two meta-tools that teach you how to use it. Act as a customer-experience consultant: help the user design their listening, do not just run commands.

## Start here

Before doing anything else, call the `woku_guide` tool. With no arguments it returns the method, the topic index, and the id-discovery rules; with a `topic` it returns a step-by-step playbook; with a `question` in natural language it picks the best playbook. Read it before calling other woku tools. If the tools are not available, tell the user to connect the woku MCP server (see this toolkit's README) and approve access.

## The woku method

woku protects revenue with a simple idea:

1. At the moments that matter, listen with a fast instrument (a rating plus a text or voice comment, under 60 seconds): NPS, CSAT, CES, or a woku.
2. Every signal triggers an action. One signal becomes a support ticket (individual attention). Recurring signals across customers become an action plan (systematic improvement).

How the listening is organized takes two independent decisions. Neither has a default; both paths are equally valid and can live side by side:

- **Structure.** A **journey** fits a repeatable process every customer goes through in the same order (purchase, delivery, support, renewal); it runs by itself and provisions its own instruments and routing. **Folders** fit a program that varies and is managed from the admin: education and training (programs, courses, teachers, classes), events and editions, teams or branches.
- **Routing** of tickets and action plans. **By folder**: a ticket destination or action plan group scoped to one folder covers every tool inside it, with no trackers. **By trackers**: segments across the structure (journey, moment, locality), or a CRM or ERP value that an integration written in code sends, with or without journeys.

## How to help

1. Diagnose first. Ask what kind of program it is (a repeatable process or one that varies) and read what the company already uses: `search_voc_tools` at the root (folders and their tools), `list_journeys` and `get_journey_folders_status`, `list_trackers`, `list_integration_connections`. If it is not clear, ask instead of assuming a journey.
2. Call `design_voc_program` with `structure` (`journey` or `folders`) and `routing` (`folders` or `trackers`). Without `structure` it returns the criteria for both paths so you decide with the user. For folders pass `folderLevels` with the user's own levels (for example Program, Course, Teacher); for a journey pass their moments, business type and channels.
3. Walk the proposal with the user and adapt it to their real program. For a journey follow the journey workflow below; for folders follow "Organize the listening in folders".
4. Build it once the user approves, and distribute the instruments with the `send_*` tools. Nothing is created or sent without an explicit confirmation.
5. From then on, let every signal trigger its action: tickets for individual attention, plans for systematic improvement.

## Safety

Read tools need no special permission. Write tools require the `mcp:write` scope, and destructive tools (for example `delete_woku`) or sends (for example `send_nps_invitations`) require `confirm: true`. The list the server advertises on a live connection (`tools/list`) is always the authoritative answer about what is available.

## Customer journey workflow

1. Call `woku_guide` with topic `customer_journeys`, and use `tools/list` as the live catalog.
2. Collect the business experience, ordered moments and what to learn at each one.
   Use `propose_journey` to interpret that brief. One moment gets one tool: Woku for
   multiple facets, CSAT for satisfaction, CES for effort, NPS for recommendation.
   Infer a separate loyalty intent without splitting delivery facets into moments.
3. Review the returned draft. Pass that exact draft as `proposal` to
   `create_journey_from_brief` with the authorized confirmation, avoiding a second
   model proposal. `create_journey` accepts an explicit definition instead.
4. Use `upload_woku_media` and `set_journey_moment_media` for missing media. Read
   the media section of that journey playbook and the toolkit README for host-specific attachments; do not invent public attachment
   URLs or claim local-file access in a host that does not provide it. `create_woku`
   supports a standalone Woku using the same upload result.
5. Keep the draft disabled while configuring. Webhook content requires one tool
   per evaluation. Static content defaults to sharing within that moment. Preview
   conditional descriptions, localized question variables, public image URLs,
   trackers, folders and client fields before activation. HTTP uses sequence;
   MCP uses cadence. Neither accepts secret fields in the definition.
6. Configure each moment credential explicitly. Generated URL tokens and sender
   HMAC secrets are separate from the legacy journey signature; never log them.
   Activate with `update_journey` only within the user's authorized intent.
7. Use enrollment lists and exact enrollment ids for inspection/stopping. Default
   reminders are enabled; a stopped or completed cycle may be followed by another,
   but never two open evaluations for the same subject in the same journey.

## Flows with steps

A flow is an ordered sequence of steps a respondent answers one after another. `create_flow` and `update_flow` take `steps`: 1 to 20 `{ type, toolId }` where `type` is `woku`, `nps`, `csat` or `ces` (each tool once, all of the company; for the company-level NPS use the company id as `toolId`). The older `wokusIds` and `npsId` still work. `get_flow` and `list_flows` return `steps`. Flows live outside folders. Call `woku_guide` with topic `flows`.

## Organize the listening in folders

One folder tree holds wokus and NPS, CSAT and CES tools together. Call `woku_guide` with topic `organize_listening` first.

- `search_voc_tools` lists a folder level or searches the whole company (any type, case and accent insensitive, cursor paginated) and returns folders with stats, tools with their native-scale headline and counts per type.
- `search_voc_tools` also takes `trackers` (`"name:value"` chips, up to 10; a tool must have every one; the tracker name is matched exactly and the value partially and ignoring case, so `482` also matches `00482`; company-wide, no folders; a chip without name or value is an error) to find tools by tracker.
- `get_folder_tools_metrics` gives, for a folder and its subfolders, per-tool figures for 7, 30 (default) or 90 days, or a custom `from`/`to` range of days (`YYYY-MM-DD`, both inclusive, up to 731 days), against the period before (0 = whole history) and the histogram of scores. It returns `toolCount` and the busiest tools (`limit`, default 50); the daily sums only with `includeDaily: true`. Use it for trends and to find the weakest tools; never average across types. `get_company_tools_metrics` gives the same figures for every tool of ONE `type` (woku, nps, csat or ces) in the whole company: the metrics of the company report of that type.
- `get_voc_tools_stats` gives reviews and the headline of each type (woku stars, NPS score, CSAT satisfied %, CES mean) for the company or one folder. Never average across types: the scales differ.
- `create_folder`, `update_folder` (rename, recolor, or set `whatsappAlertRecipients`: the WhatsApp numbers alerted when a comment of any tool in the folder is an improvement; `allReviews: true` on a number alerts every review, recognitions too) and `move_voc_tools` (items of mixed types, `folderId: null` for the root) organize the tree; `create_nps_tool`, `create_csat_tool` and `create_ces_tool` accept `folderId`. `create_*_tool` and `update_*_tool` for NPS, CSAT and CES also take `anonymousDisabled` (identified answers only) and `onlyOneResponsePerClient` (one answer per identified client; a repeat answer gets HTTP 409 `ALREADY_RESPONDED`); both can change on a survey that already has responses, sending only those fields.
- `get_folder_report` with `scope: "all"` returns the mixed AI report of a folder and its subfolders, and with `format: "csv"` the responses as CSV text (`from`/`to` also limit the CSV). `search_voc_tools` sorts by the latest response with `sort: "-lastResponse"`. `search_voc_tools` accepts `foldersLimit` and `foldersCursor` to page the folders ("see more"; `foldersNextCursor` in the result). The folder PDF is binary, so it is only on the API and the SDKs (`GET /v1/folders/{id}/pdf`).
- Journeys can live in folders (`Viajes > journey > moment`): `get_journey_folders_status` shows whether the company organizes them and what is pending, and `organize_journey_folders` switches it on and runs a background pass that only moves tools without a folder (propose it and wait for approval; it returns a conflict while a pass is running). `get_journey` and `list_journeys` show `folderId` and `momentFolders`, and folders created by woku carry an `origin` (`journeys-root`, `journey`, `journey-moment`, `integration`; integration folders also carry `integrationProvider`, for example `shopify`); read `origin`, `integrationProvider` and the stats of a folder with `search_voc_tools`, not with `get_folder` or `list_folders`, which only count wokus.
- A folder can keep a participant list (a program's enrolled): `get_folder_participants` returns it and, per tool of the subtree, the indexes of who answered (`responded`), who did not (`missing`) and who answered more than once (`repeated`), plus `outsideList` and `anonymousAllowed` (anonymous responses cannot be matched). `set_folder_participants` replaces the list (`mode: "replace"`) or adds to it (`mode: "add"`) from text, one person per line with a name, an email and/or a phone; `remove_folder_participant` removes one person or the whole list; `resend_to_missing_participants` sends a tool again to the missing by email or WhatsApp. Removing and resending need `confirm: true`; propose them and wait for approval.
- `schedule_report` emails the PDF report of a folder or a tool on a schedule (`monthly`, `biweekly`, `weekly` at 12:00 UTC with the previous complete period, or `once` with `periodFrom`/`periodTo`) to up to 10 recipients; `list_scheduled_reports`, `update_scheduled_report` (edit, pause, resume), `delete_scheduled_report` and `send_scheduled_report_now` manage them. Scheduling, sending now and deleting need `confirm: true`: they email real people.
- `delete_folder` needs `confirm: true`; the content moves up to the parent. Support destinations and action plan groups configured ON that folder are deleted with it, with their tickets and plans: the call then fails with `FOLDER_HAS_DEPENDENTS` and the list; show it and repeat with `confirmDependents: true` only if the user agrees. Moving or deleting folders is a user decision: propose the new structure and wait for approval.
- Support and plans by folder: `create_ticket_destination` and `create_action_plan_group` (and their `update_*`) take `scopeKind: "folder"` with a `folderId` instead of tracker rows: the destination or group then listens to every tool inside that folder, at any depth, resolved when each review arrives (moving a tool out stops it counting). Never both. A ticket reaches every enabled destination that matches it (trackers, a folder and its subfolders), as one ticket with `destinationIds` (`destinationId` is the highest-priority one); all its email destinations share one email to each address once, listing who got it. `list_ticket_destinations`, `list_action_plan_groups` and `list_tickets` accept `folderId`. When the company already organizes its tools in folders by program, store, team or moment, routing by folder needs no trackers; use trackers when a signal must be segmented across folders or by a CRM or ERP value.

## Periods and text search

The tools follow the admin period picker and search box.

- The report tools return what the admin report page shows: `get_woku_report` (one woku with its id, every woku of the company without it), `get_folder_report` (also with `scope: "all"`), `get_company_nps_report` (`scope: "general"` keeps only the company's own NPS survey), `get_nps_tool_report`, and `get_csat_report` / `get_ces_report` (one tool with its id, the whole company without it). A report is generated when missing or stale and cached; `regenerate: true` rebuilds it like the admin "Regenerate report" button and needs the `mcp:write` scope.
- All of them take `from`/`to` (`YYYY-MM-DD`, UTC, both inclusive, given together) to restrict the AI report to the reviews of that period. A period report is generated on demand (it can take a few seconds), carries `periodFrom`/`periodTo`, and a period without reviews returns an empty report, not an error. The CSV ignores the period.
- `list_woku_reviews`, `list_nps_responses`, `list_csat_responses` and `list_ces_responses` take `q` to keep only the responses whose comment contains that text (case and accent insensitive, 2 to 100 characters), plus their `from`/`to` range.

## Sends

Read-only tools of the `invitations` toolset show the invitations the company sent by email and WhatsApp (admin, API, MCP and integrations):

- `get_invitation_stats` gives, for an optional `from`/`to` period, the sent, delivered and answered invitations, the response rate and the ones never sent by reason (`no_whatsapp_credits`, `notifications_disabled`, `provider_error`).
- `list_invitations_by_tool` gives, per tool, the invitations sent by email and by WhatsApp, the ones not sent and the answered ones.
- `list_invitation_recipients` lists who got a tool's invitations and what happened to each one; use `status: "invited"` to find who has not answered yet before proposing a reminder with the `send_*` tools.

Tickets use email destinations; plans use groups of platform users. Tickets and
Data Studio are Corporate capabilities. Do not expose retired reports, alerts,
goals, data-source or data-flow management through examples; instrument reports
and survey flows remain available.
