# WhatsApp Flow – Human & AI Explanation

This file explains how the WhatsApp lead intake workflow works.

It is written for:
- humans who want to understand the automation
- AI tools that need project context
- future debugging and workflow changes

The exact AI prompt is stored separately in:

```text
prompts/whatsapp-agent.md
```

---

## 1. Purpose of the WhatsApp workflow

The WhatsApp workflow collects lead information from a potential property buyer.

The goal is to:

```text
Receive WhatsApp message
→ identify the customer
→ check Airtable for existing data
→ ask only the next missing question
→ save/update the customer when complete
→ send the correct WhatsApp reply
```

This workflow is used for lead qualification before the financing process continues.

---

## 2. Main workflows involved

There are two workflows involved:

```text
WhatsApp Agent - Main - DEV
Save Qualified Lead - Sub - DEV
```

### WhatsApp Agent - Main - DEV

This is the main conversation workflow.

It receives the WhatsApp message, prepares the customer context, calls the AI Agent, and sends the WhatsApp reply.

### Save Qualified Lead - Sub - DEV

This is the saving workflow.

It receives clean lead data from the main workflow and either:

```text
updates an existing Airtable customer
or
creates a new Airtable customer
```

---

## 3. Simple visual overview

```text
Customer sends WhatsApp message
        ↓
Webhook receives message
        ↓
Ignore own messages and group messages
        ↓
Search customer in Airtable by phone
        ↓
Build AI context
        ↓
AI Agent decides next reply and lead data
        ↓
Parse AI JSON
        ↓
Is lead complete?
        ↓
No  → send next question on WhatsApp
Yes → call Save Qualified Lead sub-workflow
        ↓
Save/update Airtable
        ↓
Send final WhatsApp reply
```

---

## 4. Detailed step-by-step flow

### Step 1 – Webhook receives WhatsApp message

Node:

```text
Webhook
```

Purpose:

```text
Receives incoming WhatsApp messages from WAHA.
```

The webhook receives data such as:

```text
message text
sender phone / WhatsApp ID
session
message ID
sender name
fromMe status
media status
```

---

### Step 2 – Respond quickly to webhook

Node:

```text
Respond to Webhook
```

Purpose:

```text
Immediately tells WAHA/n8n that the message was received.
```

Example response:

```json
{ "status": "received" }
```

This avoids webhook timeout problems.

---

### Step 3 – Normalize incoming WhatsApp data

Node:

```text
Data
```

Purpose:

```text
Extracts important values from the raw webhook body.
```

Important extracted fields:

```text
event
session
message_id
from
to
source
body
hasMedia
notifyName
fromMe
```

Important field:

```text
from = customer WhatsApp ID / phone identifier
```

This field is later used to search the customer in Airtable.

---

### Step 4 – Ignore own messages and group messages

Node:

```text
Ignore own messages / groups?
```

Purpose:

```text
Stops the workflow if the message was sent by your own WhatsApp account
or if the message comes from a group chat.
```

Why this is important:

```text
Prevents the bot from replying to itself.
Prevents unwanted replies in WhatsApp groups.
```

---

### Step 5 – Search customer in Airtable

Node:

```text
Search customer by Phone
```

Purpose:

```text
Checks whether this WhatsApp sender already exists in Airtable.
```

Search logic:

```text
Find Airtable customer where Phone = WhatsApp sender ID
```

In DEV, this should search the DEV table:

```text
customers_dev
```

In PROD, this should search the PROD table:

```text
customers_prod
```

---

### Step 6 – Build AI context from Airtable

Node:

```text
Build AI Context from Airtable
```

Purpose:

```text
Combines the incoming WhatsApp message with existing Airtable data.
```

It prepares context for the AI Agent:

```text
userMessage
phone
session
notifyName
customerExists
existingData
missingFields
firstMissingField
```

Important behavior:

```text
If the customer already exists, the AI should not ask again for filled fields.
The AI should ask only the first missing field.
```

Example:

```text
Existing data:
First Name = Maria
Email = maria.becker@example.com

Missing fields:
nationality, age, netSalary

AI should ask only for nationality first.
```

---

## 5. AI Agent behavior

Node:

```text
AI Agent
```

Purpose:

```text
Decides what message should be sent to the customer
and returns structured lead data as JSON.
```

The exact prompt is stored in:

```text
prompts/whatsapp-agent.md
```

The AI Agent must:

```text
reply in the same language as the customer
ask only one question at a time
preserve existing Airtable data
convert translated answers into fixed English Airtable values
return only valid JSON
```

Important:

```text
The AI response is not sent directly to Airtable.
It is first parsed and cleaned by the next node.
```

---

## 6. Fields the AI Agent collects

The AI Agent collects these fields:

```text
fullName
firstName
email
nationality
age
maritalStatus
numberChildren
netSalary
budget
timeline
mortgageStatus
propertyType
preferredAreas
```

The AI should ask in this order:

```text
1. fullName
2. firstName
3. email
4. nationality
5. age
6. maritalStatus
7. numberChildren
8. netSalary
9. budget
10. timeline
11. mortgageStatus
12. propertyType
13. preferredAreas
```

---

## 7. Fixed Airtable values

Even if the customer answers in German or French, the saved values must use fixed English values.

### Marital Status

```text
Single
Married
Divorced
Widowed
Unknown
```

### Timeline

```text
0-3 months
3-6 months
6-12 months
12+ months
Just exploring
```

### Mortgage Status

```text
No mortgage yet - needs help
Mortgage in principle
Mortgage approved
Already has mortgage
Unknown
```

### Property Type

```text
Apartment
House
Detached house
Semi-detached house
Plot of land
Commercial
Other
```

Example conversions:

```text
célibataire → Single
marié → Married
Wohnung → Apartment
appartement → Apartment
besoin d’aide → No mortgage yet - needs help
```

---

## 8. AI JSON output

The AI Agent must return only JSON.

Expected structure:

```json
{
  "replyText": "The WhatsApp message to send to the customer",
  "leadComplete": false,
  "leadData": {
    "phone": "WhatsApp ID from internal context",
    "fullName": "",
    "firstName": "",
    "email": "",
    "nationality": "",
    "age": 0,
    "maritalStatus": "Unknown",
    "numberChildren": 0,
    "netSalary": 0,
    "budget": 0,
    "timeline": "Just exploring",
    "mortgageStatus": "Unknown",
    "propertyType": "Other",
    "preferredAreas": "",
    "qualifiedLead": false
  }
}
```

---

## 9. Parse and clean AI JSON

Node:

```text
Parse AI JSON / Lead Data
```

Purpose:

```text
Reads the AI JSON output and cleans the data before saving.
```

This node:

```text
extracts JSON from the AI response
converts salary and budget to numbers
keeps existing Airtable values if the AI does not repeat them
checks whether the lead is complete
sets final clean fields for the next workflow
```

Important:

```text
This node protects the workflow from incomplete or messy AI output.
```

---

## 10. Check whether lead is complete

Node:

```text
IF Lead Complete?
```

Purpose:

```text
Decides whether the customer data is complete enough to save as a qualified lead.
```

If lead is incomplete:

```text
Send next WhatsApp question
```

If lead is complete:

```text
Call Save Qualified Lead sub-workflow
```

---

## 11. If lead is incomplete

Nodes:

```text
Start Typing - Next Question
Wait 5 seconds - Next Question
Send WhatsApp - Next Question
```

Purpose:

```text
Makes the WhatsApp reply feel more natural.
```

The customer receives the next question from:

```text
replyText
```

Example:

```text
Merci Maria. Quelle est votre nationalité ?
```

---

## 12. If lead is complete

Node:

```text
Save Qualified Lead - Sub
```

Purpose:

```text
Calls the sub-workflow that saves or updates the customer in Airtable.
```

Important:

```text
The main workflow handles conversation.
The sub-workflow handles saving.
```

This is cleaner because saving logic is separated from conversation logic.

---

## 13. Save Qualified Lead sub-workflow

Workflow:

```text
Save Qualified Lead - Sub - DEV
```

This sub-workflow:

```text
receives lead data from the main workflow
searches Airtable by Phone
checks if customer exists
updates existing customer
or creates new customer
```

---

## 14. Update existing customer

Node:

```text
Update existing customer
```

Purpose:

```text
Updates an existing Airtable row.
```

Important rule:

```text
Do not change Original Source when updating an existing customer.
```

For WhatsApp contact, update:

```text
Last Contact Channel = WhatsApp
```

Also update the collected lead fields:

```text
Name
First Name
Email
Phone
Nationality
Age
Marital Status
Number of Children
Net Salary
Budget
Timeline
Mortgage Status
Property Type
Preferred Areas
Qualified Lead
Pipeline Stage
```

For DEV workflow:

```text
Environment = dev
Table = customers_dev
```

For PROD workflow:

```text
Environment = prod
Table = customers_prod
```

---

## 15. Create new customer

Node:

```text
Create new customer
```

Purpose:

```text
Creates a new Airtable row when no customer was found.
```

For a new WhatsApp lead, set:

```text
Original Source = WhatsApp
Last Contact Channel = WhatsApp
```

For DEV workflow:

```text
Environment = dev
Table = customers_dev
```

For PROD workflow:

```text
Environment = prod
Table = customers_prod
```

---

## 16. Final WhatsApp reply

Nodes:

```text
Start Typing - Final Reply
Wait 5 seconds - Send Final Reply
Send WhatsApp - Final Reply
```

Purpose:

```text
Sends the final confirmation message after the lead has been saved.
```

Example:

```text
Thank you. We have received your information. One of our agents will contact you shortly.
```

---

## 17. Airtable table rules

The project uses two Airtable tables:

```text
customers_dev
customers_prod
```

Simple rule:

```text
DEV workflow  → customers_dev
PROD workflow → customers_prod
```

Environment field:

```text
customers_dev  → Environment = dev
customers_prod → Environment = prod
```

Source tracking:

```text
Original Source = first channel where the lead came from
Last Contact Channel = latest channel used
```

For WhatsApp:

```text
Original Source = WhatsApp only when creating a new customer
Last Contact Channel = WhatsApp every time WhatsApp is used
```

---

## 18. GitHub and workflow versioning

Do not mainly edit complex n8n workflows manually in JSON.

Recommended process:

```text
1. Create feature branch from dev
2. Make changes visually in n8n DEV workflow
3. Test with customers_dev
4. Export workflow JSON
5. Replace JSON file locally
6. Commit and push feature branch
7. Pull request: feature → dev
8. Pull request: dev → main
9. Import main version into n8n PROD workflow
```

Simple meaning:

```text
n8n DEV = where you build/test
GitHub = where you save/review versions
n8n PROD = where the stable workflow runs live
```

---

## 19. Common mistakes to avoid

```text
Do not edit PROD workflow directly.
Do not send DEV tests to customers_prod.
Do not send PROD customers to customers_dev.
Do not delete Airtable fields before checking n8n mappings.
Do not rename Airtable fields without updating n8n nodes.
Do not ask multiple questions in one WhatsApp message.
Do not let AI save translated values into Airtable select fields.
```

---

## 20. Short summary

```text
Webhook receives WhatsApp message
Data node cleans message
Airtable search finds existing customer
Context node finds missing fields
AI Agent asks one missing question
Parser cleans AI JSON
Incomplete lead gets next question
Complete lead goes to save sub-workflow
Sub-workflow updates or creates Airtable row
WhatsApp sends final reply
```

Most important rule:

```text
Conversation logic stays in WhatsApp Agent - Main.
Saving logic stays in Save Qualified Lead - Sub.
```
