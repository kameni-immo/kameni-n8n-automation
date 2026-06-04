# System IDs

Important:
- Work only on DEV by default.
- Never edit, activate, publish, or overwrite PROD unless the user explicitly says: "Apply this to PROD".
- Do not store API keys, tokens, passwords, or secrets in this file.

---

## n8n Workflows

### DEV workflows

#### WhatsApp
| Purpose | Workflow name | Workflow ID | Notes |
|---|---|---|---|
| WhatsApp lead intake | WhatsApp Lead Ingestion - DEV | 1A68dQXM9FG7BGV66f0RG | Main WhatsApp workflow |
| Save/update customer | Save Qualified Lead - Sub - DEV | mqJcNmPVa9fB56hYD7Kcg | Sub-workflow called by main workflow |

#### Phone
| Purpose | Workflow name | Workflow ID | Notes |
|---|---|---|---|
| Phone lead intake | Phone Lead Ingestion - DEV | 0pmVB4ONklkXJIP0 | Main Phone workflow |


### PROD workflows

#### WhatsApp
| Purpose | Workflow name | Workflow ID | Notes |
|---|---|---|---|
| WhatsApp lead intake | WhatsApp Lead Ingestion - PROD | VXAOQEcp5Kl0L1XI | Do not edit unless explicitly approved |
| Save/update customer | Save Qualified Lead - Sub - PROD | wjnVy8UBko5YuwI6 | Do not edit unless explicitly approved |

#### Phone
| Purpose | Workflow name | Workflow ID | Notes |
|---|---|---|---|
| Phone lead intake | Phone Lead Ingestion - PROD | DbNVde47X8HJcfnu | Do not edit unless explicitly approved |


---

## Airtable

Important:
- Current setup: DEV and PROD share the same Airtable base.
- Environment separation currently happens through different tables inside that base.
- In the future, DEV and PROD may be split into separate Airtable bases.

### DEV Airtable

| Purpose | Name | ID |
|---|---|---|
| Base | Financing Automation - DEV | appn65GylpcWvByIP |
| Customers table | Customers | tblbtkyGl7pSOpClC |


### PROD Airtable

| Purpose | Name | ID |
|---|---|---|
| Base | Financing Automation - PROD | appn65GylpcWvByIP |
| Customers table | Customers | tbl41GSX4LX7ZmGIm |


---

## Telnyx

### DEV

| Purpose | Name | ID / Value | Notes |
|---|---|---|---|
| AI phone assistant | Financing Phone Agent - DEV | assistant-3896d294-b584-483f-806f-09de4c94c8ca | Use for testing only |
| Outbound caller number | Telnyx DEV number | +12018841021 | Used as From number for outbound calls |

### PROD

| Purpose | Name | ID / Value | Notes |
|---|---|---|---|
| AI phone assistant | Financing Phone Agent - PROD | TBD | Do not modify unless explicitly approved |
| Outbound caller number | Telnyx PROD number | TBD | Do not modify unless explicitly approved |
