---
name: Import a store catalog and order history into Metrilo
description: Bulk-load customers, categories, products, and orders into a Metrilo project in the required order.
api: openapi/metrilo-tracking-openapi.yml
operations: [batchCustomers, batchCategories, batchProducts, batchOrders]
---

# Import a store catalog and order history into Metrilo

Use this to seed a Metrilo project (analytics + CRM) with an existing store's data.

## Auth
- Send the project **API Token** in every request body as `token` (Settings -> Installation).
- Every backend endpoint here requires the **`X-Digest`** header: `HMAC-SHA256(raw_request_body, API_Secret)`.
- Base URL: `https://trk.mtrl.me/v2`. All calls are `POST application/json` with the envelope `{ time, token, params }`.

## Steps (order matters)
1. **`batchCustomers`** — POST `/customer/batch` with `params` = array of customers (each needs `email`; `tags` merge, not overwrite).
2. **`batchCategories`** — POST `/category/batch` with `params` = array of categories (each needs `id`).
3. **`batchProducts`** — POST `/product/batch` with `params` = array of products (each needs `id`; `categories` is an array of category ids). Send deleted products first if applicable.
4. **`batchOrders`** — POST `/order/batch` with `params` = array of orders (each needs `id`; `email` links the customer, `products[]` uses `productId`/`quantity`).

## Rules
- Keep each request under **5MB**; page large catalogs into multiple batches.
- Import categories before products, products before orders (references resolve forward).
- Ingestion is async — allow ~1 minute before data appears in reports.
- No idempotency key: re-sending the same `id`/`email` upserts in place (see `conventions/metrilo-conventions.yml`).
- On `401` check the token/`X-Digest`; on `403` the signature is invalid or the IP is ignored (see `errors/metrilo-problem-types.yml`).
