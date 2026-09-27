---
name: woku-sdk-python
description: "Build a server-side woku integration in Python with the official woku package. Use when the user wants to manage their woku account from a Python backend: create trackers and VoC tools (NPS, CSAT, CES), send surveys by email or WhatsApp, read responses and delivery, curate support tickets, or drive action plans in code. Sync and async clients over httpx. This is server-only (secret key); for capturing feedback in a mobile app use @wokuapp/react-native instead."
---

# woku SDK for Python

The `woku` package is the official **server-side** client for the woku
management API (`/v1`), over `httpx`. Use it when the user wants woku operations
in their Python backend instead of raw HTTP. It offers a synchronous client
(`Woku`) and an asynchronous twin (`AsyncWoku`), typed request models, automatic
protected write retries and pagination. It is the counterpart of the
JavaScript SDK (`@wokuapp/sdk`), with the same surface.

**Server-only.** The company secret key grants full management access. Keep it on
the backend, never in a client you do not control. For capturing feedback from a
mobile app, use `@wokuapp/react-native` instead.

## Install and initialize

```bash
pip install woku
```

Requires Python 3.9+. Create one client and reuse it. The key comes from
`WOKU_API_KEY` when you omit `api_key`; the user gets it from the company
Information section at admin.woku.app. Request bodies accept a plain dict (as
below) or a generated Pydantic model from `woku._generated.models`.

```python
from woku import Woku

woku = Woku(api_key="sk_...")  # or set WOKU_API_KEY and call Woku()
```

## Core operations

GETs and protected writes retry with a stable key: tracker/VoC definitions,
invitations, and journey create/enroll/stop/mint-URL/event operations. Other writes
and uploads are attempted once. A key alone does not make a write idempotent.

```python
# Tracker definitions and values (wire your CRM/ERP ids to feedback tools).
tracker = woku.trackers.create({"name": "Store #1", "system": "retail"})
woku.trackers.assign_to_woku("woku_123", {"name": "Store #1", "value": "TX-42"})

# VoC tools: nps_tools / csat_tools / ces_tools (create/list/get/update/delete).
tool = woku.nps_tools.create(
    {"name": "Post-purchase", "npsMessage": "our company",
    "audienceType": "a friend or colleague"}
)

# Send a survey. IMPORTANT: `channel` is required and `recipients` is a list of
# bare strings (emails for email, phone numbers for whatsapp), NOT objects.
woku.nps.send_invitations(
    {"channel": "email", "npsToolId": tool["_id"], "recipients": ["ana@example.com"]}
)

# Read responses and delivery/response-rate.
for r in woku.nps.list_responses():
    print(r)
delivery = woku.dispatches.stats({"channel": "email"})

# Support tickets (AI-generated): list, filter, curate.
for t in woku.tickets.list({"severity": "high"}):
    print(t["title"])

# Action plans: approve and manage inside woku (change status, work tasks).
woku.action_plans.approve("plan_123")
woku.action_plans.complete("plan_123")
```

Namespaces: `trackers`, `nps_tools` / `csat_tools` / `ces_tools`, `nps` / `csat`
/ `ces`, `wokus`, `forms`, `flows`, `action_plans`, `action_plan_groups`,
`tickets`, `ticket_destinations`, `dispatches`, `reports`, `company`,
`quarantines`.

## Async

Use `AsyncWoku` for awaitable methods and `async for` iteration; use it as a
context manager to close the connection pool.

```python
import asyncio
from woku import AsyncWoku


async def main() -> None:
    async with AsyncWoku(api_key="sk_...") as woku:
        async for ticket in await woku.tickets.list({"severity": "high"}):
            print(ticket["title"])


asyncio.run(main())
```

## Pagination, errors, retries

- Sync list methods return an iterable page: iterate items with `for x in page`,
  or walk pages with `page.has_next_page()` and `page.get_next_page()`. The async
  client iterates with `async for`.
- Every failure is a `WokuError`. HTTP errors are typed subclasses
  (`NotFoundError`, `RateLimitError`, `AuthenticationError`, ...) carrying
  `status`, the parsed body and the server `request_id`. Transport failures are
  `WokuConnectionError` / `WokuTimeoutError`.
- GETs and idempotent writes retry automatically with backoff, honoring the
  `Retry-After` header. Per-call overrides go in the `options` argument.

## Rotating the key

`woku.company.rotate_key()` returns a new secret key and immediately invalidates
the old one; store `result["secretKey"]`. `woku.company.revoke_key()` disables
the key.

## When to reach for the reference

For the full method list, request/response shapes and per-call options, point the
user to the SDK page at https://woku.app/docs/en/development/sdk-python and the API
reference at https://woku.app/docs/en/development/api. There is an equivalent
JavaScript SDK (`@wokuapp/sdk`) documented at /development/sdk-javascript.

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

```python
with open("delivery.jpg", "rb") as image:
    media = woku.media.upload(image, filename="delivery.jpg", content_type="image/jpeg")
for enrollment in woku.journeys.iter_enrollments(journey_id, {"limit": 100}):
    print(enrollment["id"], enrollment["lifecycle"])
woku.journeys.preview_moment(journey_id, "delivery", {"order": "case-123", "late": True})
```

AsyncWoku provides the corresponding awaitable methods and async iterators.
