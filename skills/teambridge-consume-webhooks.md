---
name: Consume and verify Teambridge webhooks
description: Verify HMAC-signed Teambridge webhook deliveries, dedupe by event_id, and fetch the changed record.
api: openapi/teambridge-openapi-original.json
operations: [getRecord]
---

# Consume and verify Teambridge webhooks

Teambridge POSTs outbound webhooks to your HTTPS endpoint when collection records change. Configure subscriptions under Settings > Account > Outbound webhooks.

## Verify the signature (required)
1. Read headers `X-Webhook-Timestamp` (Unix seconds) and `X-Webhook-Signature` (`sha256=<hex>`).
2. Reject if `|now - timestamp| > 300` seconds (replay protection).
3. Strip the `sha256=` prefix. Compute `HMAC-SHA256(secret, "{timestamp}.{rawBody}")` using the RAW request body string (not re-serialized JSON).
4. Compare with a constant-time comparison; reject on mismatch.

## Process the event
5. Dedupe on `event_id` (UUID) — process each event at most once (idempotent handling).
6. Read `event_type` (snake_case, e.g. `shift_created`, `user_updated`, `location_deleted`) and `data.action` (`created`/`updated`/`deleted`).
7. To get full record data, call `getRecord` — `GET /v1/collections/{data.collection_id}/records/{data.record_id}` with an OAuth bearer token.

## Rules
- Return 2xx quickly; a non-2xx/500 response causes Teambridge to retry delivery.
- Payload `version` is currently `1.0`; branch on it if it changes.
