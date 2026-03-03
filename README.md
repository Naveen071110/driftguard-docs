# DriftGuard
> Schema drift & data contract guardrails API.  
> Catch breaking schema changes **before** they break dashboards and pipelines.

---

## What is DriftGuard?

DriftGuard is a small, focused API that compares an **incoming schema**
against an **expected contract** and returns a clear verdict:

- `pass` – schema matches contract
- `warning` – changes that might cause issues downstream
- `breaking` – changes that will break downstream consumers

Typical failures it prevents:

- Required column removed (`email` disappears)
- Column type changed (`timestamp → string`, `integer → string`)
- New unexpected columns added to a strict contract
- Schema deviates from what your pipelines, dashboards, or consumers expect

Instead of adopting a heavy "data observability platform", teams can
drop DriftGuard into their existing CI / pipelines in a few lines of code.

---

## This repository

This repo contains the **public documentation** for DriftGuard.

- Live docs: [**https://docs.driftguard.dev**](https://docs.driftguard.dev)
- Product site / waitlist: [**https://driftguard.dev**](https://driftguard.dev)

Docs are built with **[Mintlify](https://mintlify.com/)** and deployed
automatically from this repository.

---

## Key concepts

- **Contracts**  
  JSON definitions of your expected schema: column names and types.  
  Valid column types: `string`, `integer`, `number`, `boolean`, `date`, `timestamp`, `json`.

- **Checks**  
  One evaluation of an incoming schema against a contract.  
  Each check returns a result (`pass` / `warning` / `breaking`), a severity score (0–100), and a diff.  
  Each check consumes **1 credit**.

- **Severity levels**  
  Changes are scored and classified into three outcomes:  
  `0–19` → `pass` | `20–89` → `warning` | `90–100` → `breaking`

- **Schema Inference**  
  Send a sample JSON payload to `/api/v1/contracts/infer` and DriftGuard will
  auto-generate a contract schema for you — no manual column mapping needed.

You can read the full explanations in:

- [`concepts/contracts.mdx`](./concepts/contracts.mdx)
- [`concepts/checks.mdx`](./concepts/checks.mdx)
- [`concepts/severity-levels.mdx`](./concepts/severity-levels.mdx)
- [`concepts/rules.mdx`](./concepts/rules.mdx)

---

## Quickstart (API)

> Short version – enough for someone landing on the GitHub page.
```bash
# 1) Set your API key (format: dg_live_ or dg_test_ + 24 chars minimum)
export DRIFTGUARD_KEY="dg_live_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

# 2) (Optional) Infer a contract schema from a sample payload
curl -s -X POST https://api.driftguard.dev/api/v1/contracts/infer \
  -H "X-API-Key: $DRIFTGUARD_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "sample": {
      "user_id": 1,
      "email": "user@example.com",
      "created_at": "2026-01-01T00:00:00Z"
    }
  }'

# 3) Create a contract
curl -s -X POST https://api.driftguard.dev/api/v1/contracts \
  -H "X-API-Key: $DRIFTGUARD_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "users_production",
    "schema": {
      "columns": [
        { "name": "user_id",    "type": "integer",   "required": true },
        { "name": "email",      "type": "string",    "required": true },
        { "name": "created_at", "type": "timestamp", "required": true }
      ]
    }
  }'

# 4) Run a schema check (consumes 1 credit)
curl -s -X POST https://api.driftguard.dev/api/v1/check \
  -H "X-API-Key: $DRIFTGUARD_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contract_id": "your_contract_id_here",
    "incoming_schema": {
      "columns": [
        { "name": "user_id",    "type": "integer" },
        { "name": "created_at", "type": "string" }
      ]
    }
  }'

# Expected response
# {
#   "result": "breaking",
#   "severity": 95,
#   "diff": {
#     "removed": ["email"],
#     "type_changes": [{ "column": "created_at", "from": "timestamp", "to": "string" }],
#     "added": []
#   }
# }

# 5) Check your remaining credits
curl -s https://api.driftguard.dev/api/v1/usage \
  -H "X-API-Key: $DRIFTGUARD_KEY"
```

---

## Authentication

All endpoints (except `GET /api/v1/health`) require an API key passed via the `X-API-Key` header:
```
X-API-Key: dg_live_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Key rules:
- Format: `dg_live_<24+ chars>` or `dg_test_<24+ chars>`
- Rate limit: **100 requests/minute** per key
- Key creation limit: **10 keys/hour** per user
- Keys are shown **once** at creation — store them immediately
- Manage keys at [driftguard.dev/dashboard](https://driftguard.dev/dashboard)

---

## API Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET`  | `/api/v1/health` | Health check (no auth required) |
| `POST` | `/api/v1/contracts` | Create a new contract |
| `GET`  | `/api/v1/contracts` | List your contracts |
| `GET`  | `/api/v1/contracts/{id}` | Get a single contract |
| `POST` | `/api/v1/contracts/infer` | Infer a contract schema from sample data |
| `POST` | `/api/v1/check` | Run a schema check (costs 1 credit) |
| `GET`  | `/api/v1/checks/{id}` | Retrieve a historical check result |
| `GET`  | `/api/v1/usage` | View credits remaining + total checks run |
| `POST` | `/api/v1/webhooks` | Register a webhook (HTTPS only) |
| `GET`  | `/api/v1/webhooks` | List webhooks |
| `DELETE` | `/api/v1/webhooks/{id}` | Delete a webhook |
| `GET`  | `/api/v1/keys` | List API keys |
| `POST` | `/api/v1/keys` | Create a new API key |
| `POST` | `/api/v1/keys/{id}/revoke` | Deactivate a key |
| `POST` | `/api/v1/keys/{id}/activate` | Reactivate a key |
| `DELETE` | `/api/v1/keys/{id}` | Hard delete a key (must be revoked first) |

Full API reference: [**docs.driftguard.dev/api-reference**](https://audionique-6f0b3793.mintlify.app/api-reference)
