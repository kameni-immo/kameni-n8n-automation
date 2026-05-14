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

## ElevenLabs

### DEV agents

| Purpose | Agent name | Agent ID | Notes |
|---|---|---|---|
| Phone qualification agent | Financing Phone Agent - DEV | agent_0801khgdz2jeek0r874z6zcw875g | Use for testing only |

### PROD agents

| Purpose | Agent name | Agent ID | Notes |
|---|---|---|---|
| Phone qualification agent | Financing Phone Agent - PROD | agent_5801krkqj6w8f48bdn02j64e8fdn | Do not modify unless explicitly approved |