---
name: provision-cloud-instance
description: Pick a region and plan, create a compute instance, control its power state, and delete it - with the rate limits and the irreversible steps called out.
api: akamai-technologies:linode-api
operations:
  - get-regions
  - get-linode-types
  - get-images
  - post-linode-instance
  - get-linode-instance
  - get-linode-instances
  - post-boot-linode-instance
  - post-shutdown-linode-instance
  - post-reboot-linode-instance
  - delete-linode-instance
spec: openapi/akamai-technologies-linode-api-openapi.json
generated: '2026-08-30'
method: generated
source: openapi/akamai-technologies-linode-api-openapi.json + https://techdocs.akamai.com
---

# Provision and manage an Akamai Cloud (Linode) instance

## Steps

1. `get-regions` for placement, `get-linode-types` for plans and prices, `get-images` for the OS image.
2. `post-linode-instance` with `region`, `type`, `image`, `label` and either `root_pass` or `authorized_keys`.
   The response returns `id`, `ipv4` and `status: "provisioning"`.
3. Poll `get-linode-instance` until `status` is `running`.
4. Power control: `post-boot-linode-instance`, `post-shutdown-linode-instance`, `post-reboot-linode-instance`.
5. `delete-linode-instance` destroys the instance **and its disks**. There is no undo; recover only from a backup you
   already had (`post-restore-backup`).

## Rate limits

Documented and worth respecting: 1,600 requests/minute by default, but only **200 requests/minute for paginated GET
collections** - which is exactly what a polling agent hits first. Poll a single instance by id rather than re-listing.

## Pagination and filtering

`page` and `page_size`; the response carries `data`, `page`, `pages` and `results`. Filter server-side with the
`X-Filter` header rather than fetching every page.

## Authentication

Bearer personal access token, or OAuth 2.0 authorization code via `https://login.linode.com/oauth/authorize`. Scope it:
`linodes:read_only` is enough to inspect, `linodes:read_write` to create or destroy. Full scope list in
`scopes/akamai-technologies-scopes.yml`.
