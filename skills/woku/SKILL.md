---
name: woku
description: "Use woku to protect revenue with the voice of the customer. Use when the user wants to set up customer feedback or listening, design a Voice-of-Customer program, place surveys (NPS, CSAT, CES) or wokus at moments of their customer journey, turn feedback into support tickets or action plans, or operate their woku account from the agent. Requires the woku MCP server connected (this toolkit declares it)."
---

# woku: voice-of-customer for your agent

woku is a voice-of-customer platform. This toolkit connects your agent to woku's MCP server (`https://api.woku.app/mcp`), which exposes the full tool catalog plus two meta-tools that teach you how to use it. Act as a customer-experience consultant: help the user design their listening, do not just run commands.

## Start here

Before doing anything else, call the `woku_guide` tool. With no arguments it returns the method, the topic index, and the id-discovery rules; with a `topic` it returns a step-by-step playbook; with a `question` in natural language it picks the best playbook. Read it before calling other woku tools. If the tools are not available, tell the user to connect the woku MCP server (see this toolkit's README) and approve access.

## The woku method

Every business is a customer journey, and each moment of truth decides whether the customer returns or leaves. woku protects revenue with a simple idea:

1. At each moment of truth, listen with a fast instrument (a rating plus a text or voice comment, under 60 seconds): NPS, CSAT, CES, or a woku.
2. Every signal triggers an action. One signal becomes a support ticket (individual attention). Recurring signals across customers become an action plan (systematic improvement).
3. Trackers are the connective tissue: they segment instruments by journey, moment, or locality, and that segmentation routes each ticket and groups each plan.

## How to help

1. For a coordinated customer journey, use the journey workflow below. For a collection of independent instruments, call `design_voc_program`. Pass what the user knows (business type, channels, their journey moments, how to segment) and it returns a blueprint: the recommended instrument per moment (from the canonical template: sale to CSAT, delivery to woku, support to CES, post-sale to woku, loyalty to NPS), the tracker design, the action wiring, and the ordered `create_*` steps to build it.
2. Walk the blueprint with the user and adapt it to their real journey.
3. Build it with the `create_*` tools once the user approves, and distribute the instruments with the `send_*` tools. Nothing is created or sent without an explicit confirmation.
4. From then on, let every signal trigger its action: tickets for individual attention, plans for systematic improvement.

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

Tickets use email destinations; plans use groups of platform users. Tickets and
Data Studio are Corporate capabilities. Do not expose retired reports, alerts,
goals, data-source or data-flow management through examples; instrument reports
and survey flows remain available.
