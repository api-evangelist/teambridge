---
name: Create records and sync external mappings
description: Create a record in a Teambridge collection and link it to an external system (Bullhorn, ADP) via mappings.
api: openapi/teambridge-openapi-original.json
operations: [createRecord, listMappings, createMapping, updateMapping, deleteMapping]
---

# Create records and sync external mappings

Write data into Teambridge and keep it linked to an external system of record.

## Auth
OAuth 2.0 client credentials from `https://teambridge.us.auth0.com/oauth/token` (scope `write`); send `Authorization: Bearer <token>`.

## Steps
1. `createRecord` — `POST /v1/collections/{collectionId}/records` with the record body conforming to the collection's fields (discover them first with `listFields`). Returns the new record and its `recordId`.
2. `createMapping` — `POST /v1/mappings` to link the Teambridge record to an external record (e.g. Bullhorn, ADP).
3. `listMappings` — `GET /v1/mappings` to review existing external links.
4. `updateMapping` / `deleteMapping` — `PUT`/`DELETE /v1/mappings/{mappingId}` to change or remove a link.

## Rules
- Envelope `{ "data": ..., "error": ... }`; a 400 signals validation errors — inspect `error.message`/`error.code`.
- To read back a record with its external links inline, call `listRecords` with `enriched=true`.
- IDs are UUIDs; a 404 on mapping ops means the `mappingId` does not exist.
