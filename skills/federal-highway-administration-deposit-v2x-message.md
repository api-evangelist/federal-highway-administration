---
name: deposit-v2x-message
description: >-
  Authenticate against the FHWA V2X App API and deposit a UPER-encoded SAE J2735 MAP or TIM
  message to the ETX MQTT broker, scoped by a GeoJSON geofence, then verify and withdraw the
  deployment.
api: FHWA V2X App API
api_id: federal-highway-administration:v2x-app-api
spec: openapi/federal-highway-administration-v2x-app-api-openapi.json
generated: '2026-09-09'
method: generated
source: openapi/federal-highway-administration-v2x-app-api-openapi.json
operations:
  - postRegistration_1
  - createGeofence
  - deposit
  - getAllActiveGeofenceDeployments
  - getGeofenceDeployment
  - getGeofenceGeohashes
  - deleteGeofence_1
---

# Deposit a V2X message

The V2X App API is **self-hosted**. There is no FHWA-operated endpoint: the contract's only
server entry is the springdoc placeholder `http://localhost:8080`. Before anything below,
establish the base URL of the instance you are talking to — the operator supplies it.

## 1. Get a token

`postRegistration_1` — `POST /auth/token`, body `TokenPostRequest {username, password}`,
returns `TokenPostResponse`. Every operation in this skill except decoding requires the
resulting Keycloak JWT as `Authorization: Bearer <token>`.

## 2. Define the geofence (optional)

`createGeofence` — `POST /prd/v2/configurations/geofence`, body `ConfigurationGeofence`.
The geometry is a GeoJSON `GeofenceFeatureCollection` (Polygon / MultiPolygon / LineString /
MultiLineString).

You can skip this step. `deposit` extracts the geofence from the message itself when
`override_geofence` is not supplied.

## 3. Deposit

`deposit` — `POST /prd/v2/deposit/geofence`, body `DepositRequest`:

- `asn1_hex` (**required**) — a UPER-encoded SAE J2735 `MessageFrame` in hex, carrying a MAP
  or a TIM.
- `override_geofence` (optional) — a `GeofenceFeatureCollection` that replaces the geofence
  the message would otherwise imply.

**There is no idempotency key on this operation.** Replaying the same `asn1_hex` deposits the
message again. Retry only after confirming with step 4 that the first call did not land.

## 4. Verify

- `getAllActiveGeofenceDeployments` — `GET /prd/v2/deposit/geofence/deployments`
- `getGeofenceDeployment` — `GET /prd/v2/deposit/geofence/deployments/{geofenceId}`
- `getGeofenceGeohashes` — `GET /prd/v2/deposit/geofence/deployments/{geofenceId}/geohashes`

The geohashes are the routing cells the sidecar publishes on; the `kafka-producer` service
emits a `GeoHashRoutedMsg` protobuf per active geofence at 1 Hz.

## 5. Withdraw

`deleteGeofence_1` — `DELETE /prd/v2/deposit/geofence?identifier=<id>` takes the deployment
back. **The contract states no window** — it does not say how long a deposit remains
withdrawable. A separate expiration service reaps expired deployments on an interval the
operator configures (`GEOFENCE_CLEANUP_INTERVAL`, sample default `5m`), which is a deployment
setting and not a guarantee. If your caller needs a hard deadline, get it from the operator,
not from this contract.

## Errors

Two envelopes, neither RFC 9457:

- `ErrorResponse {error, description}` — application errors.
- `KeycloakErrorResponse {timestamp, status, error, path}` — the auth layer.

Declared codes across the surface: 400, 401, 403, 404, 409, 413, 422, 500. **No 429 is
declared anywhere and no rate-limit headers exist**, so back off on 5xx by elapsed time, not
by a `Retry-After` you will not receive.
