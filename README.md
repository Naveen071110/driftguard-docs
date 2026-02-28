# DriftGuard

> Schema drift & data contract guardrails API.  
> Catch breaking schema changes **before** they break dashboards and pipelines.

---

## What is DriftGuard?

DriftGuard is a small, focused API that compares an **incoming schema**
against an **expected contract** and returns a clear verdict:

- `OK` – schema matches contract
- `WARNING` – changes that might cause issues
- `BREAKING` – changes that will break downstream consumers

Typical failures it prevents:

- Required column removed (`email` disappears)
- Column type narrowed (`timestamp → string`, `string → integer`)
- Nullability changed in a breaking way
- New columns added where they shouldn’t be

Instead of adopting a heavy “data observability platform”, teams can
drop DriftGuard into their existing CI / pipelines in a few lines of code.

---

## This repository

This repo contains the **public documentation** for DriftGuard.

- Live docs: [**https://docs.driftguard.dev**](https://docs.driftguard.dev)  
  (Replace with your Mintlify URL while in beta)
- Product site / waitlist: [**https://driftguard.dev**](https://driftguard.dev) (placeholder)

Docs are built with **[Mintlify](https://mintlify.com/)** and deployed
automatically from this repository.

---

## Key concepts

- **Contracts**  
  JSON definitions of your expected schema: column names, types,
  nullability, and evaluation rules.

- **Checks**  
  One evaluation of an incoming schema against a contract.
  Each check returns a status + diff and consumes one credit.

- **Severity levels**  
  Every change is classified as `OK`, `WARNING`, or `BREAKING`
  according to your rules.

- **Rules**  
  Contract-level options like `allow_additional_columns` and
  `treat_renames_as_breaking` to tune sensitivity.

You can read the full explanations in:

- [`concepts/contracts.mdx`](./concepts/contracts.mdx)  
- [`concepts/checks.mdx`](./concepts/checks.mdx)  
- [`concepts/severity-levels.mdx`](./concepts/severity-levels.mdx)  
- [`concepts/rules.mdx`](./concepts/rules.mdx)

---

## Quickstart (API)

> Short version – enough for someone landing on the GitHub page.

```bash
# 1) Set your API key
export DRIFTGUARD_KEY="dg_live_xxxxxxxxxxxx"

# 2) Create a contract
curl -s -X POST https://api.driftguard.dev/v1/contracts \
  -H "Authorization: Bearer $DRIFTGUARD_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "users_production",
    "columns": [
      { "name": "user_id",    "type": "string",    "nullable": false },
      { "name": "email",      "type": "string",    "nullable": false },
      { "name": "created_at", "type": "timestamp", "nullable": false }
    ],
    "rules": {
      "allow_additional_columns": false,
      "treat_renames_as_breaking": true
    }
  }'

# 3) Run a schema check
curl -s -X POST https://api.driftguard.dev/v1/checks \
  -H "Authorization: Bearer $DRIFTGUARD_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contract_id": "ctr_01HX7KMQZP",
    "incoming_schema": [
      { "name": "user_id",    "type": "string" },
      { "name": "created_at", "type": "string" }
    ],
    "source": "ci"
  }'
