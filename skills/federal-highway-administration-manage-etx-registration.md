---
name: manage-etx-registration
description: >-
  Register and connect an ETX MQTT broker client through the FHWA V2X App API, check its
  registration state, and clean up stale registrations for a vendor.
api: FHWA V2X App API
api_id: federal-highway-administration:v2x-app-api
spec: openapi/federal-highway-administration-v2x-app-api-openapi.json
generated: '2026-09-09'
method: generated
source: openapi/federal-highway-administration-v2x-app-api-openapi.json
operations:
  - postRegistration_1
  - checkRegistration
  - postRegistration
  - putRegistration
  - postConnection
  - cleanupVendorRegistrations
---

# Manage an ETX registration

## 1. Token

`postRegistration_1` — `POST /auth/token`. Bearer JWT for everything below.

## 2. Check before you write

`checkRegistration` — `GET /prd/v2/registration?DeviceID=<id>`, returns
`RegistrationCheckResponse`. Do this first: registrations are capped per vendor and per user
(see the limits skill), and a blind re-register can burn quota.

## 3. Register, then connect

- `postRegistration` — `POST /prd/v2/registration`, body `RegistrationPostRequest` →
  `RegistrationResponse`.
- `postConnection` — `POST /prd/v2/connection`, body `ConnectionPostRequest` →
  `ConnectionResponse`.

`putRegistration` — `PUT /prd/v2/registration`, body `RegistrationPutRequest` — updates an
existing registration.

### Do not use the combined call

`postRegistrationConnection` — `POST /prd/v2/registration-connection` — does both in one
step and is **marked `deprecated: true` in the contract**. FHWA states no successor and no
removal date; the two operations above are the evident replacement.

## 4. Clean up

`cleanupVendorRegistrations` — `GET /prd/v2/registration/cleanup/vendor/{vendorId}` removes
"old registrations for vendor". **The contract does not define "old"**, so do not tell a user
what will be deleted. Enumerate first with the limits and deployment read operations, and
treat this call as irreversible unless the operator has told you otherwise.

Retry parameters `retry`, `num_retries` and `num_sec_to_sleep` appear as query parameters on
parts of this surface — read them off the contract for the exact operation rather than
assuming they are present.
