---
name: woku-sdk-javascript
description: "Build a server-side woku integration in JavaScript or TypeScript with the official @wokuapp/sdk. Use when the user wants to manage their woku account from a Node.js backend: create trackers and VoC tools (NPS, CSAT, CES), send surveys by email or WhatsApp, read responses and delivery, curate support tickets, or drive action plans in code. This is server-only (secret key); for capturing feedback in a mobile app use @wokuapp/react-native instead."
---

# woku SDK for JavaScript and TypeScript

`@wokuapp/sdk` is the official **server-side** client for the woku management API
(`/v1`). Use it when the user wants woku operations in their Node.js backend
instead of raw HTTP. It is typed, retries transient failures, makes creates
idempotent, and auto-paginates lists.

**Server-only.** The company secret key grants full management access. Keep it on
the backend, never in a browser, mobile app or other untrusted client. For
capturing feedback from a mobile app, use `@wokuapp/react-native` (a public
capture key) instead.

## Install and initialize

```bash
npm install @wokuapp/sdk
```

Requires Node.js 18+. Create one client and reuse it. The key comes from
`WOKU_API_KEY` when you omit `apiKey`; the user gets it from the company
Information section at admin.woku.app.

```ts
import { Woku } from '@wokuapp/sdk';

const woku = new Woku({ apiKey: process.env.WOKU_API_KEY });
```

## Core operations

Creates are idempotent (auto `Idempotency-Key`); actions (send, test, reply) are
never auto-retried.

```ts
// Tracker definitions and values (wire your CRM/ERP ids to feedback tools).
const tracker = await woku.trackers.create({ name: 'Store #1', system: 'retail' });
await woku.trackers.assignToWoku('woku_123', { name: 'Store #1', value: 'TX-42' });

// VoC tools: npsTools / csatTools / cesTools (create/list/get/update/delete).
const tool = await woku.npsTools.create({
  name: 'Post-purchase',
  npsMessage: 'How likely are you to recommend us?',
});

// Send a survey. IMPORTANT: `channel` is required and `recipients` is an array
// of bare strings (emails for email, phone numbers for whatsapp), NOT objects.
await woku.nps.sendInvitations({
  channel: 'email',
  npsToolId: tool._id,
  recipients: ['ana@example.com'],
});

// Read responses and delivery/response-rate.
for await (const r of await woku.nps.listResponses()) console.log(r);
const delivery = await woku.dispatches.stats({ channel: 'email' });

// Support tickets (AI-generated): list, filter, curate.
for await (const t of await woku.tickets.list({ severity: 'high' })) console.log(t.title);

// Action plans: approve and manage inside woku (change status, work tasks).
await woku.actionPlans.approve('plan_123');
await woku.actionPlans.complete('plan_123');
```

Namespaces: `trackers`, `npsTools` / `csatTools` / `cesTools`, `nps` / `csat` /
`ces`, `wokus`, `forms`, `flows`, `actionPlans`, `actionPlanGroups`, `tickets`,
`ticketDestinations`, `dispatches`, `reports`, `company`, `quarantines`.

## Pagination, errors, retries

- List methods return a `Page`: iterate items with `for await (const x of page)`,
  or walk pages with `page.hasNextPage()` and `page.getNextPage()`.
- Every failure is a `WokuError`. HTTP errors are typed subclasses
  (`NotFoundError`, `RateLimitError`, `AuthenticationError`, ...) carrying
  `status`, the parsed body and the server `requestId`. Transport failures are
  `WokuConnectionError` / `WokuTimeoutError`.
- GETs and idempotent writes retry automatically with backoff, honoring the
  `Retry-After` header.

## Rotating the key

`woku.company.rotateKey()` returns a new secret key and immediately invalidates
the old one; store the returned key. `woku.company.revokeKey()` disables the key.

## When to reach for the reference

For the full method list, request/response shapes and per-call options, point the
user to the SDK page at https://docs.woku.app/development/sdk-javascript and the
API reference at https://docs.woku.app/development/api. There is an equivalent
Python SDK (`woku`) documented at /development/sdk-python.
