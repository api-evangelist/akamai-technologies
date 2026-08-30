---
name: activate-property-version
description: Create a new property version, edit its rule tree, activate it to staging then production, and cancel a pending activation if it turns out wrong.
api: akamai-technologies:papi-v1
operations:
  - get-contracts
  - get-groups
  - get-properties
  - get-property
  - get-latest-property-version
  - post-property-versions
  - get-property-version-rules
  - put-property-version-rules
  - post-property-activations
  - get-property-activation
  - delete-property-activation
spec: openapi/akamai-technologies-papi-v1-openapi.json
generated: '2026-08-30'
method: generated
source: openapi/akamai-technologies-papi-v1-openapi.json + https://techdocs.akamai.com
---

# Change and activate an Akamai property configuration

## The model

You never edit a live Akamai configuration. You create a **new version** of the property, edit that version, then
**activate** it to a network. Activation is staged and cancellable while pending, which is what makes this safe to
automate.

## Steps

1. Find the scope. `get-contracts` then `get-groups`; nearly every Property Manager collection needs `contractId` and
   `groupId` as query parameters.
2. `get-properties` to locate the property, then `get-latest-property-version` to see what is live on
   `PRODUCTION` and `STAGING`.
3. `post-property-versions` with `createFromVersion` set to the current production version. This gives you an editable
   version; the live one is untouched.
4. `get-property-version-rules` for the new version, modify the rule tree, and `put-property-version-rules` it back.
   Use `If-Match` with the ETag you received so a concurrent edit fails with `412` rather than silently overwriting.
5. `post-property-activations` with `network: "STAGING"`, the version number, and `notifyEmails`. Validate on staging -
   the staging network is real Akamai edge, not a simulator.
6. Repeat step 5 with `network: "PRODUCTION"` once staging looks right.
7. Poll `get-property-activation` until `status` leaves `PENDING`.

## Undo

- While an activation is `PENDING`: `delete-property-activation` cancels it.
- Once it has completed: you cannot cancel. Activate the previous version number instead - that is the rollback.

## Watch out

- `PAPI-Use-Prefixes` decides whether ids come back as `prp_123456` or `123456`. Set it explicitly and be consistent,
  or your ids will not match between calls.
- Production activation is not instant; fast-push products complete in minutes, others longer.
- `accountSwitchKey` is required if you are acting on a managed account rather than your own.

## Authentication

EdgeGrid EG1-HMAC-SHA256 against your per-account hostname. Requires an API client with READ-WRITE grant on the
Property Manager API and access to the relevant group.
