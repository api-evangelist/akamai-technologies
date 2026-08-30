---
name: purge-cached-content
description: Invalidate or delete cached objects by URL, CP code or cache tag on the Akamai staging or production network, choosing the reversible option by default and respecting the object rate limits.
api: akamai-technologies:ccu-v3
operations:
  - post-invalidate-url
  - post-delete-url
  - post-invalidate-cpcode
  - post-delete-cpcode
  - post-invalidate-tag
  - post-delete-tag
  - post-rate-limit-status
spec: openapi/akamai-technologies-ccu-v3-openapi.json
generated: '2026-08-30'
method: generated
source: openapi/akamai-technologies-ccu-v3-openapi.json + https://techdocs.akamai.com
---

# Purge cached content at the Akamai edge

## When to use this

The caller wants freshly published content served from the Akamai edge, or wants an object gone.

## Choose invalidate, not delete

`post-invalidate-url` marks the object **stale**: the edge keeps the copy and revalidates with origin on the next
request. If origin is down, the stale copy still serves. `post-delete-url` **evicts** the object: the next request is a
guaranteed origin fetch, and there is no way to undo it.

**Default to invalidate.** Only use delete when the content must not be served again under any circumstances (a legal
takedown, a leaked asset). This is the one irreversible write in the whole Akamai control plane.

## Steps

1. Decide the target type. URL/ARL for individual objects, CP code for an entire reporting group, cache tag for a
   logical set you tagged at the edge.
2. Check your headroom first with `post-rate-limit-status` for that purge type. The limits are per account and are
   counted in **objects**, not requests: 10,000 for URL, 5,000 for cache tag, 300 for CP code.
3. Pick the network. Path parameter `network` is `staging` or `production`. Rehearse on `staging`.
4. POST the objects array to the matching operation, e.g. `post-invalidate-url` with
   `{"objects": ["https://www.example.com/a.jpg", "https://www.example.com/b.css"]}`.
5. A `201` returns `purgeId`, `estimatedSeconds` and `supportId`. Purge is asynchronous - `estimatedSeconds` is the
   expected completion, typically about five seconds.

## Limits and errors

- Read `X-Ratelimit-Remaining` and `X-Ratelimit-Remaining-Objects` on every 201 and stop before you hit zero.
- On `429`, the RFC 7807 body carries live `rateLimit`, `rateLimitRemaining` and `rateLimitCurrentRequestSize`, and
  `X-Ratelimit-Seconds-To-Refresh-Limit` tells you exactly how long to wait. Wait that long; do not retry immediately.
- `413` means the objects array exceeds the per-request object limit. Split the batch.
- Quote `supportId` when opening a case.

## Authentication

EdgeGrid. Sign the request with EG1-HMAC-SHA256 and send it to your per-account hostname
(`https://akab-xxxxxxxx.luna.akamaiapis.net/ccu/v3`). The upstream spec does not declare this scheme - see
`overlays/akamai-technologies-ccu-v3-overlay.yaml`.
