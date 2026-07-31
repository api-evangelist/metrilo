---
name: Sync a customer and apply CRM tags
description: Create or update a customer in Metrilo and manage their CRM tags for segmentation.
api: openapi/metrilo-tracking-openapi.yml
operations: [createCustomer, tagCustomer, untagCustomer]
---

# Sync a customer and apply CRM tags

Use this to keep a single customer's Metrilo CRM record current and drive segmentation with tags.

## Auth
- Base URL: `https://trk.mtrl.me/v2`. All calls are `POST application/json` with `{ time, token, params }`.
- `token` (API Token) goes in the body on every call.
- **`createCustomer` (`POST /customer`) is the one endpoint that does NOT require `X-Digest`.**
- **`tagCustomer` / `untagCustomer` DO require `X-Digest`** = `HMAC-SHA256(raw_request_body, API_Secret)`.

## Steps
1. **`createCustomer`** — POST `/customer`, `params` = `{ email, firstName?, lastName?, phoneNumber?, subscribed?, tags? }`. `email` is the identity key; existing fields are replaced, `tags` merge.
2. **`tagCustomer`** — POST `/customer/tag`, `params` = `{ email, tags: [...] }` to add segmentation tags. Include `X-Digest`.
3. **`untagCustomer`** — POST `/customer/untag`, `params` = `{ email, tags: [...] }` to remove tags. Include `X-Digest`.

## Rules
- Tags are additive on the customer record; removing requires `untagCustomer`.
- Do not cache customer-action calls — it breaks attribution (see `conventions/metrilo-conventions.yml`).
- Errors are HTTP status codes only (`errors/metrilo-problem-types.yml`): `401` = bad token / missing `X-Digest`, `403` = bad signature / ignored IP, `402` = billing.
