---
name: woku-sdk-javascript
description: "Build a server-side woku integration in JavaScript or TypeScript with the official @wokuapp/sdk. Use when the user wants to manage their woku account from a Node.js backend: create trackers and VoC tools (NPS, CSAT, CES), send surveys by email or WhatsApp, read responses and delivery, curate support tickets, or drive action plans in code. This is server-only (secret key); for capturing feedback in a mobile app use @wokuapp/react-native instead."
---

# woku SDK for JavaScript and TypeScript

`@wokuapp/sdk` is the official **server-side** client for the woku management API
(`/v1`). Use it when the user wants woku operations in their Node.js backend
instead of raw HTTP. It is typed, retries transient failures, makes creates
safe for protected writes, and supports pagination.

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

GETs and protected writes retry with a stable key: tracker/VoC definitions,
invitations, and journey create/enroll/stop/mint-URL/event operations. Other writes
and uploads are attempted once. A key alone does not make a write idempotent.

```ts
// Tracker definitions and values (wire your CRM/ERP ids to feedback tools).
const tracker = await woku.trackers.create({ name: 'Store #1', system: 'retail' });
await woku.trackers.assignToWoku('woku_123', { name: 'Store #1', value: 'TX-42' });

// VoC tools: npsTools / csatTools / cesTools (create/list/get/update/delete).
const tool = await woku.npsTools.create({
  name: 'Post-purchase',
  npsMessage: 'our company',
  audienceType: 'a friend or colleague',
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
`ticketDestinations`, `dispatches`, `reports`, `company`, `quarantines`, `journeys`, `media`.

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
user to the SDK page at https://woku.app/docs/en/development/sdk-javascript and the
API reference at https://woku.app/docs/en/development/api. There is an equivalent
Python SDK (`woku`) documented at /development/sdk-python.

## Customer journeys and media

Prefer an actual journey when the task coordinates several business moments.
One moment has one tool: several facets use Woku, satisfaction CSAT, effort CES,
recommendation NPS. A separate loyalty intention can add a moment; several facets
of delivery must not become several delivery moments. Journey response tools are
identified. Operator sends start immediately; opening a QR/link waits for a saved
first response. Later moments wait or use their webhook, optionally with a backup.
Default wait is ten days; zero means one hour. Keep reminders enabled unless asked.

The SDK exposes 17 journey methods, plus lazy enrollment iteration. Public cursor
pages allow 100 items (default 20); the Admin page endpoint is separate and uses 50.
Use the exact enrollment id to stop one case. Editing creates a new definition
version; running cases keep their snapshot. A cycle finishes on its last response
or 30 days from the first send of the last moment.

Media uploads use multipart and return fileId for moment.toolSpec.fileId. They do
not retry automatically; 413 is PayloadTooLargeError. Company keys remain on the
backend; minted webhook URLs use a separate HTTP transport without that key.
Keep request/idempotency ids stable during uncertain retries. Deduplication lasts
24 hours; inspect the outcome before submitting with a new key.

Journey and media methods require SDK version 0.3.0 or later. Verify the installed
version and changelog when upgrading an existing integration.

```ts
const media = await woku.media.upload({ file: imageBytes, filename: 'delivery.jpg', contentType: 'image/jpeg' });
for await (const enrollment of woku.journeys.iterEnrollments(journeyId, { limit: 100 })) {
  console.log(enrollment.id, enrollment.lifecycle);
}
await woku.journeys.previewMoment(journeyId, 'delivery', { order: 'case-123', late: true });
```
