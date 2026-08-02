---
name: Read Teambridge collection records
description: Authenticate, discover collections and their fields, then list and read records with filtering and pagination.
api: openapi/teambridge-openapi-original.json
operations: [listCollections, listFields, listRecords, getRecord]
---

# Read Teambridge collection records

Use the Teambridge External API (base `https://open-api.teambridge.com`) to read workforce data (shifts, users, placements, locations, or custom collections).

## Auth
Obtain an OAuth 2.0 access token via the client-credentials flow from
`https://teambridge.us.auth0.com/oauth/token` (scope `write`). Send it as
`Authorization: Bearer <token>` on every request. A 401 means the token is missing/expired — mint a new one.

## Steps
1. `listCollections` — `GET /v1/collections` to enumerate available collections and their `collectionId` (UUID).
2. `listFields` — `GET /v1/collections/{collectionId}/fields` to learn each collection's field names/types before filtering.
3. `listRecords` — `GET /v1/collections/{collectionId}/records`. The `page` (zero-based) and `size` (1–50, default 20) query params are REQUIRED. Add up to 10 field filters using operators `_is`, `_contains`, `_gt`, `_gte`, `_lt`, `_lte`. Pass `enriched=true` to include `externalRecordMappings` (Bullhorn/ADP IDs) in each record's metadata.
4. `getRecord` — `GET /v1/collections/{collectionId}/records/{recordId}` for a single record's full data.

## Rules
- Responses use the envelope `{ "data": ..., "error": null }`; on error `data` is null and `error` holds a message or `{message, code}`.
- Date-time fields are ISO 8601 with timezone.
- Paginate by incrementing `page` until fewer than `size` records return. Never request `size` > 50 (returns 400).
