# AGENTS.md

This file provides shared instructions for Codex CLI and other AI coding tools working in this repository.

## Project Overview

This repository contains an n8n automation project for real estate and financing lead qualification.
It connects WhatsApp, phone calls, AI agents, Airtable, ElevenLabs, and related tools.

Workflows are created visually in n8n, exported as JSON, and versioned in Git.
This is a documentation-first, configuration-as-code project.

There are no build commands, package managers, or automated test runners.

## Core Rule: DEV First

Always work on DEV by default.

Never edit, activate, publish, overwrite, or delete PROD resources unless the user explicitly says:

> Apply this to PROD

Before using tools or editing references for n8n, Airtable, or ElevenLabs, check:

```text
docs/system-ids.md
```

This file contains the DEV and PROD IDs.

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

## Repository Structure

```text
workflows/       n8n workflow JSON exports
prompts/         backups of AI prompts used in n8n nodes
docs/            process documentation and system references
tests/           sample webhook payloads for manual testing
AGENTS.md        shared instructions for Codex and other AI tools
CLAUDE.md        Claude Code specific instructions
README.md        project overview
```

## Architecture

### WhatsApp Flow

The WhatsApp flow lives in:

```text
workflows/whatsapp/
```

It has two parts:

- Main workflow: receives WhatsApp/WAHA webhooks, manages conversation logic, and checks whether the lead is complete.
- Sub-workflow: saves or updates the qualified lead/customer in Airtable.

The WhatsApp AI agent should ask only one missing question at a time.

### Phone Flow

The phone flow lives in:

```text
workflows/phone/
```

It handles phone number cleanup, Airtable updates, ElevenLabs outbound calls, transcript processing, and final confirmation.

### Airtable

Airtable is the central source of truth.

Environment separation is mandatory:

- DEV workflows use DEV Airtable tables only.
- PROD workflows use PROD Airtable tables only.

Do not rename or delete Airtable fields without checking all n8n nodes and prompt files that reference them.

## Workflow Editing Rules

1. Edit workflows only in n8n DEV by default.
2. Test with DEV Airtable data.
3. Export JSON from n8n DEV.
4. Replace the matching JSON file in `workflows/`.
5. If a prompt changed in n8n, update the matching file in `prompts/`.
6. Commit changes to GitHub.
7. Promote to PROD only after explicit user approval.

## AI Prompt Rules

AI output inside n8n workflows must be raw JSON only.

Do not wrap JSON in markdown fences.
Do not add explanations before or after JSON.

Keep prompt backup files in `prompts/` synchronized with the real prompt text inside n8n nodes.

## Tool Safety Rules

When connected to external systems:

- Prefer read/list/search before write actions.
- Do not delete workflows, Airtable records, Jira issues, agents, or files unless explicitly requested.
- Do not modify PROD resources unless the user explicitly says: "Apply this to PROD".
- When unsure, explain the intended action before making changes.

## Documentation Map

- `docs/system-ids.md` — DEV/PROD IDs for n8n, Airtable, and ElevenLabs
- `docs/airtable-fields.md` — Airtable field schema and expected values
- `docs/lead-process.md` — end-to-end lead process
- `docs/phone-call-flow.md` — phone workflow explanation
- `docs/whatsapp-flow.md` — WhatsApp workflow explanation
- `prompts/` — backup of AI system prompts used inside n8n nodes

## Communication Style

When responding to the user:

- Be short and beginner-friendly.
- Explain technical things in simple words.
- Warn clearly before any PROD-impacting action.
- Prefer small, safe steps.
