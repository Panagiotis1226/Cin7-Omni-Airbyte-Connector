# Cin7 Omni Airbyte Connector

A custom Airbyte source connector (forked from the original Cin7 Connector) for [Cin7 Omni](https://www.cin7.com/), a cloud-based inventory management platform. This connector uses the [Cin7 Omni REST API](https://api.cin7.com/api) to sync data into your data warehouse.

---

## Overview

| Property | Value |
|---|---|
| **Connector Type** | Source |
| **Airbyte** |
| **Authentication** | HTTP Basic Auth |
| **Base URL** | `https://api.cin7.com/api` |
| **Rate Limits** | 3 req/sec · 60 req/min · 5,000 req/day |

---

## Prerequisites

- A Cin7 Omni account at [https://www.cin7.com/](https://www.cin7.com/)
- An **Account ID** and **API Key** from Cin7 Omni

### Getting Your API Credentials

1. Log in to your Cin7 Omni account
2. Navigate to **Integrations → API**
3. Copy your **Account ID** and **API Key**

---

## Configuration

| Field | Description | Required |
|---|---|---|
| `username` | Your Cin7 Omni **Account ID** (used as Basic Auth username) | ✅ |
| `api_key` | Your Cin7 Omni **API Key** (used as Basic Auth password) | ✅ |

---

## Supported Streams

All streams use:
- **Pagination**: Page-based (`page` + `rows` parameters), 250 records per page
- **HTTP Method**: `GET`
- **Response format**: Flat JSON array

| Stream Name | Endpoint | Primary Key |
|---|---|---|
| `contacts` | `GET /v1/Contacts` | `id` |
| `products` | `GET /v1/Products` | `id` |
| `sales_orders` | `GET /v1/SalesOrders` | `id` |
| `purchase_orders` | `GET /v1/PurchaseOrders` | `id` |
| `stock` | `GET /v1/Stock` | `productId`, `productOptionId`, `branchId` |
| `payments` | `GET /v1/Payments` | `id` |
| `credit_notes` | `GET /v1/CreditNotes` | `id` |
| `quotes` | `GET /v1/Quotes` | `id` |
| `branches` | `GET /v1/Branches` | `id` |
| `branch_transfers` | `GET /v1/BranchTransfers` | `id` |
| `adjustments` | `GET /v1/Adjustments` | `id` |
| `bom_masters` | `GET /v2/BomMasters` | `id` |
| `production_jobs` | `GET /v1/ProductionJobs` | `id` |
| `product_categories` | `GET /v1/ProductCategories` | `id` |
| `product_options` | `GET /v1/ProductOptions` | `id` |
| `serial_numbers` | `GET /v1/SerialNumbers` | `id` |
| `size_ranges` | `GET /v1/SizeRanges` | `id` |
| `users` | `GET /v1/Users` | `id` |
| `vouchers` | `GET /v1/Voucher` | `id` |

> **Note:** The `stock` stream uses a composite primary key because records do not have a top-level `id` field.

---

## Authentication Details

This connector uses **HTTP Basic Authentication**. Your Account ID is sent as the username and your API Key as the password, encoded in Base64 as per the HTTP Basic Auth standard.

You can verify your credentials with:

```bash
curl -u "YOUR_ACCOUNT_ID:YOUR_API_KEY" "https://api.cin7.com/api/v1/Contacts?rows=1"
```

---

## Rate Limiting

The Cin7 Omni API enforces the following rate limits:

| Window | Limit |
|---|---|
| Per second | 3 requests |
| Per minute | 60 requests |
| Per day | 5,000 requests |

The connector automatically handles **HTTP 429** (Too Many Requests) responses by backing off and retrying.

---

## Differences from Cin7 Core Connector

This connector is **not** compatible with the Cin7 Core (DEAR Inventory) connector. Key differences:

| Property | Cin7 Core | Cin7 Omni |
|---|---|---|
| Base URL | `https://inventory.dearsystems.com/externalapi/v2/` | `https://api.cin7.com/api/v1/` |
| Authentication | Custom headers (`api-auth-accountid`, `api-auth-applicationkey`) | HTTP Basic Auth |
| Rate limit trigger | HTTP 503 | HTTP 429 |
| Page size param | `limit` | `rows` |

---

## Notes

- All dates are in UTC format: `yyyy-MM-ddTHH:mm:ssZ`
- Schemas use `additionalProperties: true` to future-proof against API changes
- The `bom_masters` stream uses the **v2** endpoint (`/v2/BomMasters`)
- The `vouchers` stream endpoint is singular: `/v1/Voucher` (not `/v1/Vouchers`)
