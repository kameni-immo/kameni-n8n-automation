# Shared Workflows

This folder is for reusable workflow notes or shared workflow parts.

For now, the project mainly has:

```text
workflows/whatsapp/
workflows/phone/
```

Use this folder later if the same logic is reused by several workflows.

Examples of possible shared workflows:

```text
send-team-notification.json
create-customer-folder.json
validate-documents.json
send-document-request.json
generate-vorgangsnummer.json
```

Simple rule:

```text
If a workflow is used by both WhatsApp and phone workflows,
it can live here.
```

Current important shared logic idea:

```text
search customer by phone
create or update Airtable customer
send notification
create document folder
```

Do not put active workflow JSON files here unless they are really reused by multiple flows.
