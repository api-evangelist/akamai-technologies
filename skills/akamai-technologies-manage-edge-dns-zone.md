---
name: manage-edge-dns-zone
description: Use the Edge DNS change-list workflow to stage record changes, review the diff, and submit them - or throw the change list away.
api: akamai-technologies:config-dns-v2
operations:
  - get-zones
  - get-zone
  - post-zone
  - post-changelists
  - get-changelists-zone-diff
  - post-changelists-zone-recordsets
  - post-changelists-zone-submit
  - delete-changelists-zone
spec: openapi/akamai-technologies-config-dns-v2-openapi.json
generated: '2026-08-30'
method: generated
source: openapi/akamai-technologies-config-dns-v2-openapi.json + https://techdocs.akamai.com
---

# Change Akamai Edge DNS records safely

## The model

Edge DNS uses the same stage-then-commit shape as Property Manager, but the staging object is a **change list**.
Records are never edited in place.

## Steps

1. `get-zones` / `get-zone` to confirm the zone and its type (primary, secondary, alias).
2. `post-changelists` for the zone. This snapshots the current zone into an editable change list.
3. `post-changelists-zone-recordsets` (or the add-change operation) to stage record additions, edits and removals.
4. `get-changelists-zone-diff` and read it. This is the review step - it is the only place you see exactly what will
   change before it is authoritative.
5. `post-changelists-zone-submit` to make the change list live.

## Undo

Before submit: `delete-changelists-zone` discards the entire change list and nothing reaches DNS. After submit the
change is authoritative and the only fix is another change list, which is why step 4 exists.

## Watch out

- A change list is per zone and per user; a second change list on the same zone will conflict.
- DNSSEC-signed zones re-sign on submit; allow for propagation before asserting the change is visible.

## Authentication

EdgeGrid EG1-HMAC-SHA256 against your per-account hostname, with a READ-WRITE grant on the Edge DNS Zone Management API.
