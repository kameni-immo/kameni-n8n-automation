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
| WhatsApp lead intake | WhatsApp Agent - Main - DEV | 1A68dQXM9FG7BGV66f0RG | Main WhatsApp workflow |
| Save/update customer | Save Qualified Lead - Sub - DEV | mqJcNmPVa9fB56hYD7Kcg | Sub-workflow called by main workflow |

#### Phone
| Purpose | Workflow name | Workflow ID | Notes |
|---|---|---|---|
| Outbound phone lead intake (form-triggered) | Phone Lead Ingestion - DEV | 0pmVB4ONklkXJIP0 | Form → Telnyx outbound AI call → transcript → Airtable |
| Inbound phone lead intake (Telnyx webhook) | Inbound Phone Lead Ingestion - DEV | aoHOxDflf4U5VIDn | Telnyx inbound call webhook → transcript → Airtable upsert → WhatsApp confirmation |


### PROD workflows

#### WhatsApp
| Purpose | Workflow name | Workflow ID | Notes |
|---|---|---|---|
| WhatsApp lead intake | WhatsApp Agent - Main - PROD | VXAOQEcp5Kl0L1XI | Do not edit unless explicitly approved |
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

Both agents were imported from ElevenLabs and contain the same qualification prompt and French greeting.

### Phone numbers

| Purpose | Number | Phone Number ID | Notes |
|---|---|---|---|
| Outbound AI calls | +12018841021 | 2971878528225641699 | Used for DEV and PROD calls |

### TeXML application

| Purpose | App ID | Notes |
|---|---|---|
| AI call routing | 2972510149958174164 | Used in the `Telnyx Calling Agent` HTTP node |

### Webhook (Part 2 — post-call)

| Environment | URL |
|---|---|
| DEV | `https://n8n.srv1293983.hstgr.cloud/webhook/0a45c953-a3a2-4074-8df8-c37d416836bb-dev` |

Record matching: `x-telnyx-call-control-id` header in the webhook = `call_sid` stored in Airtable by Part 1.

### DEV agents

| Purpose | Agent name | Agent ID | Notes |
|---|---|---|---|
| Phone qualification agent | Phone Qualification Agent - DEV | assistant-3896d294-b584-483f-806f-09de4c94c8ca | Use for testing only |

### PROD agents

| Purpose | Agent name | Agent ID | Notes |
|---|---|---|---|
| Phone qualification agent | Phone Qualification Agent - PROD | assistant-53f0fcb1-1aa3-44a5-8d4c-124f834c8145 | Do not modify unless explicitly approved |

---

## ElevenLabs (replaced by Telnyx — do not use)

### DEV agents

| Purpose | Agent name | Agent ID | Notes |
|---|---|---|---|
| Phone qualification agent | Financing Phone Agent - DEV | agent_0801khgdz2jeek0r874z6zcw875g | Replaced by Telnyx DEV agent |

### PROD agents

| Purpose | Agent name | Agent ID | Notes |
|---|---|---|---|
| Phone qualification agent | Financing Phone Agent - PROD | agent_5801krkqj6w8f48bdn02j64e8fdn | Replaced by Telnyx PROD agent |
