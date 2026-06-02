# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an **n8n workflow automation project** for real estate and financing lead qualification.
It ingests leads via WhatsApp and phone calls, qualifies them through AI-driven conversations, and stores structured data in Airtable.

All workflows are created and edited in the n8n visual builder, exported as JSON, and versioned in Git.

There are no build commands, package managers, or automated test runners. This is a documentation-first, configuration-as-code project.

## Most Important Rule: DEV First

We only develop and edit **DEV workflows** by default.

Never edit, activate, publish, overwrite, or delete **PROD workflows**, **PROD Airtable tables**, or **PROD ElevenLabs agents** unless the user explicitly says:

> Apply this to PROD

Before using n8n, Airtable, or ElevenLabs MCP tools, always check:

```text
docs/system-ids.md
```

Use this file to distinguish DEV and PROD workflow IDs, Airtable base/table IDs, and ElevenLabs agent IDs.

If an ID is missing or unclear, stop and ask the user before changing anything.

## External Systems and MCP Usage

This project may use MCP servers for:

- n8n workflow access
- Airtable database access
- ElevenLabs voice agent access
- GitHub repository access
- Atlassian/Jira task access

Use MCP tools carefully. Prefer reading and inspecting before writing.

Do not create duplicate workflows, duplicate Airtable tables, or duplicate ElevenLabs agents if an existing DEV resource already exists in `docs/system-ids.md`.

Never store API keys, tokens, passwords, or secrets in repository files.

## Secrets Rule

Never store secrets in this repository.

Do not write or commit:

- API keys
- access tokens
- passwords
- cookies
- private credentials
- webhook secrets

IDs such as n8n workflow IDs, Airtable base/table IDs, and ElevenLabs agent IDs may be documented in `docs/system-ids.md` because they are references, not secrets.

## Tool Safety Rules

When connected to external systems:

- Prefer read/list/search before write actions.
- Do not delete workflows, Airtable records, Jira issues, agents, or files unless explicitly requested.
- Do not modify PROD resources unless the user explicitly says: "Apply this to PROD".
- When unsure, explain the intended action before making changes.

## Architecture

### Two-Channel Lead Intake

**WhatsApp flow** (`workflows/whatsapp/`):
A two-workflow system. The main workflow handles conversation logic — receiving WAHA webhooks, querying Airtable for existing data, building AI context, sending the next qualifying question.

When all required fields are collected, it calls the sub-workflow (`save-qualified-lead-sub.json`) which handles Airtable upsert logic.

**Phone flow** (`workflows/phone/`):
A two-part workflow triggered by form submission.

Part 1 cleans the phone number via AI, creates an Airtable row, sends a pre-call SMS, then starts an ElevenLabs outbound call.

Part 2 receives the transcript webhook, extracts structured data via AI, updates Airtable, and sends a confirmation SMS.

## Airtable as Central Hub

Airtable is the single source of truth.

Every interaction starts with a phone-number lookup to find or create a customer row.

The schema is documented in:

```text
docs/airtable-fields.md
```

Do not rename or delete fields without checking all n8n workflow nodes that reference them.

Environment isolation:

- `customers_dev` is used by DEV n8n workflows only
- `customers_prod` is used by PROD n8n workflows only

Use the exact Airtable base and table IDs documented in `docs/system-ids.md`.

## AI Integration Pattern

Every AI node follows this pattern:

```text
input data → n8n template expression → AI prompt → structured JSON output → JSON parser → next step
```

AI output must always be raw JSON with no markdown wrapping.

Each AI prompt has a corresponding backup file in:

```text
prompts/
```

These prompt files must be kept in sync with the actual n8n node content.

## Webhook Safety Convention

Both workflows respond to webhooks immediately before long processing starts.

Reason: this avoids timeout errors.

Processing continues asynchronously after the webhook response is sent.

## Key Conventions

### Language vs. Database Values

AI agents may detect the customer's language and respond in that language.

However, values saved to Airtable must use fixed English values.

Examples:

- Save `Single`, not `Célibataire`
- Save `Apartment`, not `Appartement`
- Save `House`, not `Maison`

Prompts must explicitly enforce this rule.

### One Question at a Time

The WhatsApp agent asks only one missing field per message.

Never ask multiple qualification questions in one WhatsApp message.

### Source Tracking

`Original Source` is set once on first contact and must not be overwritten.

`Last Contact Channel` is updated on every interaction.

### leadComplete Flag

The WhatsApp AI sets:

```json
{"leadComplete": true}
```

only when all required qualifying fields are present.

The workflow branches on this flag:

- `false` → continue the conversation
- `true` → trigger the save sub-workflow

## Development Workflow

```text
1. Edit workflows in n8n DEV visual builder only
2. Test against customers_dev table with sample webhook data from tests/
3. Export workflow JSON from n8n DEV
4. Replace the corresponding JSON file in workflows/
5. If a prompt changed, update the corresponding file in prompts/
6. Commit to GitHub
7. Only after explicit user approval, import or apply changes to n8n PROD
```

The sample files in `tests/` are copy/paste payloads for manual webhook testing inside n8n.
They are not automated tests.

## PROD Safety Rules

Do not do any of the following unless the user explicitly says `Apply this to PROD`:

- edit PROD workflows
- activate or deactivate PROD workflows
- publish PROD workflows
- overwrite PROD workflow JSON
- delete PROD workflows
- modify PROD Airtable schema
- delete PROD Airtable records
- modify PROD ElevenLabs agents
- trigger live customer-facing PROD calls or messages

When unsure, work in DEV only.

## File and Folder Conventions

```text
workflows/        n8n workflow JSON exports
workflows/whatsapp/ WhatsApp workflow exports
workflows/phone/ Phone workflow exports
docs/             project documentation and system IDs
prompts/          backup copies of AI prompts used inside n8n nodes
tests/            manual test payloads
AGENTS.md         shared AI tool instructions
CLAUDE.md         Claude Code specific instructions
README.md         project overview
```

## Documentation Map

- `docs/system-ids.md` — DEV and PROD IDs for n8n, Airtable, and ElevenLabs
- `docs/airtable-fields.md` — full field schema with types, select options, and examples
- `docs/lead-process.md` — end-to-end process map covering both channels and DEV/PROD rules
- `docs/phone-call-flow.md` — numbered step-by-step breakdown of the phone workflow
- `docs/whatsapp-flow.md` — numbered step-by-step breakdown of the WhatsApp workflow
- `prompts/` — backup of all AI system prompts used in n8n nodes; headers indicate which node and workflow each belongs to

## Documentation Sync Rules

Whenever a change is made to any part of this project, update the corresponding `.md` file(s) **in the same task** before finishing. Do not leave docs out of sync.

| What changed | Update this file |
|---|---|
| WhatsApp workflow logic or node order | `docs/whatsapp-flow.md` |
| Phone workflow logic or node order | `docs/phone-call-flow.md` |
| Overall lead process or channel routing | `docs/lead-process.md` |
| Any AI prompt inside an n8n node | Matching file in `prompts/` |
| Airtable field added, renamed, deleted, or option changed | `docs/airtable-fields.md` |
| New n8n workflow created or deleted | `docs/system-ids.md` |
| New Airtable base or table created | `docs/system-ids.md` |
| New ElevenLabs agent created | `docs/system-ids.md` |
| New folder or file convention introduced | `CLAUDE.md` and `AGENTS.md` (File and Folder Conventions section) |
| New external system or MCP integration added | `CLAUDE.md`, `AGENTS.md`, and `docs/lead-process.md` as applicable |
| README project description changes | `README.md` |

### How to apply this rule

After completing any change:

1. Identify which row(s) in the table above apply.
2. Open the listed file(s).
3. Update only the parts that reflect the change — do not rewrite the whole file.
4. Briefly note in your response which docs were updated and what changed in them.

If a change spans multiple areas (e.g., a new Airtable field that is also referenced in a prompt and a workflow), update all affected files.

## Testing Rule

After implementing any task, always verify it works before reporting completion or moving to the next task.

Steps to follow after every implementation:

1. Show the user exactly how to test the change — specific steps, not generic advice.
2. Execute or walk through the test yourself where possible (trigger a DEV webhook, check an Airtable row, verify an n8n execution log).
3. Report the result clearly: what passed, what the output was, and any issues found.
4. Do not move to the next task until the user confirms the test passed.

General testing checklist for n8n workflow changes:

```text
1. Trigger the workflow with a test payload (use files in tests/ where available)
2. Check the n8n execution log — no errors, all nodes green
3. Open customers_dev in Airtable and verify the expected fields are populated
4. If the workflow sends a message (WhatsApp, email), confirm it was received with correct content
```

Never report a task as done if:

- The n8n execution log shows errors
- Airtable was not updated as expected
- The expected customer-facing message was not received

## Working Style

Work in small steps.

Prefer inspecting existing workflows and docs before changing anything.

When changing workflows, explain briefly:

- what changed
- why it changed
- which DEV workflow/file was affected
- what docs were updated to reflect the change
- what should be tested next

Do not make broad changes across many workflows unless the user asks for it.