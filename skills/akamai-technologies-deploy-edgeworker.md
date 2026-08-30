---
name: deploy-edgeworker
description: Register an EdgeWorker, activate a version to staging then production, and roll back to the previous active version when something goes wrong.
api: akamai-technologies:edgeworkers-v1
operations:
  - get-ids
  - post-ids
  - get-id
  - post-activations
  - get-activations
  - get-activation
  - delete-activation
  - post-rollback-to-previous-active-version
  - post-deactivations
spec: openapi/akamai-technologies-edgeworkers-v1-openapi.json
generated: '2026-08-30'
method: generated
source: openapi/akamai-technologies-edgeworkers-v1-openapi.json + https://techdocs.akamai.com
---

# Deploy and roll back an Akamai EdgeWorker

## Steps

1. `get-ids` to list EdgeWorker IDs, or `post-ids` to register a new one against a `groupId` and resource tier.
2. Upload the code bundle as a version (multipart tgz), then `post-activations` with the version and
   `network: "STAGING"`.
3. Poll `get-activation` until the activation completes.
4. Repeat `post-activations` with `network: "PRODUCTION"`.

## Undo - this API has the best reversal story on the platform

- Activation still in progress: `delete-activation` cancels it.
- Version already live and misbehaving: `post-rollback-to-previous-active-version` restores the previously active
  version on that network. This is a single call and does not require you to remember the old version number.
- Want the EdgeWorker off the network entirely: `post-deactivations`.

## Watch out

- EdgeWorkers are bound to traffic by a Property Manager rule behaviour, so a rollback here does not change which
  property routes to the EdgeWorker - only which code runs.
- Report IDs 2 and 4 were disabled on 2026-08-12; check the EdgeWorkers changelog before relying on a report id.

## Authentication

EdgeGrid EG1-HMAC-SHA256 against your per-account hostname.
