---
name: woku
description: "Use woku to protect revenue with the voice of the customer. Use when the user wants to set up customer feedback or listening, design a Voice-of-Customer program, place surveys (NPS, CSAT, CES) or wokus at moments of their customer journey, turn feedback into support tickets or action plans, or operate their woku account from the agent. Requires the woku MCP server connected (this toolkit declares it)."
compatibility: Claude Code, Claude Desktop, Cursor, Codex, OpenCode
maintainer: woku
metadata:
  author: woku
  version: "0.1.0"
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

1. To design a program, call `design_voc_program`. Pass what the user knows (business type, channels, their journey moments, how to segment) and it returns a blueprint: the recommended instrument per moment (from the canonical template: sale to CSAT, delivery to woku, support to CES, post-sale to woku, loyalty to NPS), the tracker design, the action wiring, and the ordered `create_*` steps to build it.
2. Walk the blueprint with the user and adapt it to their real journey.
3. Build it with the `create_*` tools once the user approves, and distribute the instruments with the `send_*` tools. Nothing is created or sent without an explicit confirmation.
4. From then on, let every signal trigger its action: tickets for individual attention, plans for systematic improvement.

## Safety

Read tools need no special permission. Write tools require the `mcp:write` scope, and destructive tools (for example `delete_woku`) or sends (for example `send_nps_invitations`, `send_report_now`) require `confirm: true`. The list the server advertises on a live connection (`tools/list`) is always the authoritative answer about what is available.
