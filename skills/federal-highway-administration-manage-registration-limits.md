---
name: manage-registration-limits
description: >-
  Read and set the per-vendor and per-user ETX registration limits that the FHWA V2X App API
  enforces, using the admin surface.
api: FHWA V2X App API
api_id: federal-highway-administration:v2x-app-api
spec: openapi/federal-highway-administration-v2x-app-api-openapi.json
generated: '2026-09-09'
method: generated
source: openapi/federal-highway-administration-v2x-app-api-openapi.json
operations:
  - getAllVendorLimits
  - getVendorLimitsByVendorId
  - createOrUpdateVendorLimits
  - deleteVendorLimits
  - getAllUserLimits
  - getUserLimitsByVendorId
  - getUserLimitsByUsername
  - createOrUpdateUserLimits
  - deleteUserLimits
---

# Manage registration limits

These are **registration quotas**, not HTTP rate limits. They cap how many ETX registrations
a vendor or a user may hold. The API declares no request-rate limiting at all: no 429, no
`RateLimit-*` headers, no `Retry-After`.

## Vendor limits

- `getAllVendorLimits` — `GET /prd/v2/admin/vendor-limits`
- `getVendorLimitsByVendorId` — `GET /prd/v2/admin/vendor-limits/{vendorId}`
- `createOrUpdateVendorLimits` — `POST /prd/v2/admin/vendor-limits`, body
  `VendorLimitsRequest` → `VendorLimitsResponse`
- `deleteVendorLimits` — `DELETE /prd/v2/admin/vendor-limits/{vendorId}`

## User limits

- `getAllUserLimits` — `GET /prd/v2/admin/user-limits`
- `getUserLimitsByVendorId` — `GET /prd/v2/admin/user-limits/vendor/{vendorId}`
- `getUserLimitsByUsername` — `GET /prd/v2/admin/user-limits/user/{username}`
- `createOrUpdateUserLimits` — `POST /prd/v2/admin/user-limits`, body `UserLimitsRequest`
- `deleteUserLimits` — `DELETE /prd/v2/admin/user-limits/user/{username}/vendor/{vendorId}`

## Safety notes

- The two `createOrUpdate*` calls are **upserts keyed on identity**, so a repeat converges
  rather than duplicating. That is a property of these operations, not an idempotency
  guarantee — the API declares no `Idempotency-Key` header anywhere.
- Both deletes are reversible only by re-issuing the matching `createOrUpdate*` with the
  previous values. **Read the current limits and keep them before you delete.** The contract
  states no undo window.
- No collection endpoint here paginates. `getAllVendorLimits` and `getAllUserLimits` return
  unbounded arrays; size your context accordingly.
