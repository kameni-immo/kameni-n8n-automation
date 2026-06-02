# Phone Call Flow – Human & AI Explanation

This file explains how the phone-call lead intake workflow works.

It is written for:
- humans who want to understand the automation
- AI tools that need project context
- future debugging and workflow changes

The exact transcript extraction prompt is stored separately in:

```text
prompts/phone-transcript-extraction.md
```

---

## 1. Purpose of the phone-call workflow

The phone-call workflow starts with a lead form, creates a customer in Airtable, sends an SMS, starts an AI phone call, receives the call transcript, extracts structured lead data, updates Airtable, and sends a final confirmation SMS.

Simple goal:

```text
Form submitted
→ create customer
→ send SMS
→ start AI call
→ receive transcript
→ extract lead data
→ update Airtable
→ send confirmation SMS
```

---

## 2. Workflow name

```text
Data Ingestion Kameni - Finanzierung - DEV
```

This workflow currently has two main parts:

```text
Part 1 = form submission → outbound AI call
Part 2 = Telnyx post-call webhook → Airtable update → SMS confirmation
```

---

## 3. Workflow diagrams

### Part 1 – Form submission to outbound call

```mermaid
flowchart LR
    FRM(["On form submission"]) --> OPT{"If Opt-In?"}
    OPT -->|"No - stop"| STOP(["End"])
    OPT -->|"Yes - continue"| LLM["Basic LLM Chain<br/>clean phone + extract first name"]
    LLM --> CRE["Create a record<br/>Airtable customers_dev"]
    CRE --> SMS["Send a text message<br/>pre-call WhatsApp via WAHA"]
    SMS --> WT["Wait 30s"]
    WT --> TL["Telnyx Calling Agent<br/>Telnyx outbound call"]
    TL --> UPD["Update record<br/>save Call SID + Call Status"]

    subgraph ai1 ["LLM Chain sub-nodes"]
        direction LR
        OAI["OpenAI Chat Model<br/>gpt-4.1-mini"]
        SOP["Structured Output Parser"]
    end
    OAI -.->|"ai_languageModel"| LLM
    SOP -.->|"ai_outputParser"| LLM
```

### Part 2 – Transcript webhook to confirmation SMS

```mermaid
flowchart LR
    WH(["Webhook<br/>Telnyx post-call transcript"]) --> RW["Respond to Webhook"]
    RW --> FR["Format Response<br/>extract structured data from transcript"]
    FR --> UPD1["Update record1<br/>Airtable via Conversation ID"]
    UPD1 --> LLM1["Basic LLM Chain1<br/>generate confirmation SMS"]
    LLM1 --> SMS1(["Send a text message1<br/>final SMS to customer"])

    subgraph ai2 ["LLM Chain sub-nodes"]
        direction LR
        OAI1["OpenAI Chat Model1<br/>gpt-4.1-mini"]
        SOP1["Structured Output Parser1"]
        OAI2["OpenAI Chat Model2<br/>gpt-4.1-mini"]
    end
    OAI1 -.->|"ai_languageModel"| FR
    SOP1 -.->|"ai_outputParser"| FR
    OAI2 -.->|"ai_languageModel"| LLM1
```

---

## 4. Part 1 – Form submission and call start

### Step 1 – Form is submitted

Node:

```text
On form submission
```

Purpose:

```text
Receives the first customer information from a simple form.
```

Collected fields:

```text
Customer Name
Email
Phone Number
Opt in
```

---

### Step 2 – Check Opt-In

Node:

```text
If Opt-In
```

Purpose:

```text
Continues only if the customer selected Yes.
```

Simple rule:

```text
Opt in = Yes → continue
Opt in = No  → stop
```

This protects the workflow from calling customers who did not agree.

---

### Step 3 – Clean phone number and first name

Node:

```text
Basic LLM Chain
```

Purpose:

```text
Formats the phone number and extracts the customer first name.
```

The AI receives:

```text
Customer Phone Number
Customer First Name / Customer Name
Customer Email Address
```

It returns:

```text
Customer Phone Number in E.164 format
Customer First name
Customer Email Address
```

Example:

```text
0049 151 23456789 → +4915123456789
Maria Becker → Maria
```

---

### Step 4 – Create Airtable customer

Node:

```text
Create a record
```

Purpose:

```text
Creates the first Airtable row before the call starts.
```

DEV workflow should write to:

```text
customers_dev
```

PROD workflow should write to:

```text
customers_prod
```

Recommended field mapping:

```text
Name = Customer Name
First Name = extracted first name
Email = customer email
Phone = cleaned E.164 phone number
Opt In = true
Environment = dev or prod
Pipeline Stage = Qualifying
Original Source = Form
Last Contact Channel = Form
Age = 0
Number of Children = 0
Net Salary = 0
Budget = 0
```

Important:

```text
If the call is started because the customer submitted a form,
Original Source should be Form.
```

---

### Step 5 – Send pre-call WhatsApp message

Node:

```text
Send a text message
```

Type: WAHA (WhatsApp)

Purpose:

```text
Sends the customer a WhatsApp message telling them an AI agent will call shortly.
```

Example meaning:

```text
Bonjour Maria, merci pour votre inscription à notre appel de qualification ! Notre agent IA vous appellera très prochainement pour en savoir plus sur votre situation. Veuillez accepter l'appel et répondre aux questions de qualification.
```

---

### Step 6 – Wait before calling

Node:

```text
Wait
```

Purpose:

```text
Waits 30 seconds before starting the call.
```

Why:

```text
Gives the customer a small pause after receiving the SMS.
```

---

### Step 7 – Start Telnyx outbound call

Node:

```text
Telnyx Calling Agent
```

Purpose:

```text
Starts the outbound AI phone call through the Telnyx TeXML AI calls API.
```

It sends:

```text
From = +12018841021 (Telnyx phone number)
To = customer phone in E.164 format
AIAssistantId = assistant-3896d294-b584-483f-806f-09de4c94c8ca (DEV)
dynamic_variables.customer_phone = customer phone in E.164 format
```

The `customer_phone` dynamic variable is injected into the Telnyx AI assistant's context so the agent can include it in the post-call webhook payload for record matching.

Authentication:

```text
Bearer token via TELNYX_API_KEY environment variable in n8n.
Do not store API keys directly inside exported workflow JSON.
```

Endpoint:

```text
POST https://api.telnyx.com/v2/texml/ai_calls/2972510149958174164
```

---

### Step 8 – Save call SID in Airtable

Node:

```text
Update record
```

Purpose:

```text
Updates the Airtable row with the call identifiers from the Telnyx response.
```

Fields saved:

```text
Call SID = Telnyx call SID from response
Call Status = Initiated
```

Note:

```text
Conversation ID is NOT available in the initial call response.
It is received in the post-call webhook (Part 2).
```

---

## 5. Part 2 – Transcript webhook and lead extraction

### Step 9 – Transcript webhook receives call result

Node:

```text
Webhook
```

Purpose:

```text
Receives the post-call data from the Telnyx AI assistant's webhook tool after the conversation ends.
```

Webhook URL (DEV):

```text
https://n8n.srv1293983.hstgr.cloud/webhook/0a45c953-a3a2-4074-8df8-c37d416836bb-dev
```

Expected data includes:

```text
call_summary   = full Q&A summary generated by the Telnyx agent
phone_number   = customer phone in E.164 format (from dynamic variable customer_phone)
```

Note:

```text
The payload is sent by the Telnyx AI assistant using its webhook tool (send-lead-to-n8n),
triggered by the post-conversation instructions. It is NOT the built-in
post_conversation_settings webhook (that does not support webhook_url via API).
```

---

### Step 10 – Respond quickly to webhook

Node:

```text
Respond to Webhook
```

Purpose:

```text
Confirms that the transcript was received.
```

Example response:

```json
{
  "status": "received",
  "message": "Transcript accepted for processing"
}
```

---

### Step 11 – Extract structured data from transcript

Node:

```text
Format Response
```

Purpose:

```text
Reads the call summary sent by the Telnyx agent and extracts the customer's answers.
```

Input field:

```text
$json.body.call_summary  (plain text Q&A summary from the Telnyx webhook tool)
```

The exact prompt is stored in:

```text
prompts/phone-transcript-extraction.md
```

The AI extracts:

```text
nationality
age
maritalStatus
numberChildren
netSalary
budgetRange
timeline
mortgageStatus
propertyType
preferredAreas
qualifiedLead
```

Important:

```text
The AI should extract only the customer's answers,
not the agent's questions.
```

---

## 6. Fixed values for transcript extraction

The extracted values must match Airtable select values exactly.

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

### Call Status

```text
Not Initiated
Initiated
Answered
Completed
Failed
No Answer
```

---

## 7. Update customer after transcript

Node:

```text
Update record1
```

Purpose:

```text
Updates the customer row using the extracted transcript data.
```

Matching field:

```text
Call SID  (matched against $('Webhook').item.json.headers['x-telnyx-call-control-id'])
```

Why this works: the call_sid returned by the Telnyx AI calls API in Part 1 is identical to the x-telnyx-call-control-id header Telnyx sends in Part 2 webhook requests. Both sides already have this value — no dynamic variables needed.

Recommended update fields:

```text
Age
Budget
Call Status = Completed
Marital Status
Mortgage Status
Nationality
Net Salary
Number of Children
Phone (used for matching)
Pipeline Stage = Awaiting Documents
Preferred Areas
Property Type
Qualified Lead
Timeline
```

Important:

```text
Do not change Original Source during transcript update.
Only update Last Contact Channel.
```

---

## 8. Generate confirmation SMS

Node:

```text
Basic LLM Chain1
```

Purpose:

```text
Creates a short friendly SMS confirming the customer information was saved.
```

Current behavior:

```text
Generates a French confirmation SMS.
```

Example meaning:

```text
Hello Maria, your information has been saved. We will contact you with the next steps for your financing request.
```

---

## 9. Send final WhatsApp message

Node:

```text
Send a text message1
```

Type: WAHA (WhatsApp)

Purpose:

```text
Sends the final confirmation WhatsApp message to the customer.
```

The message is sent to the phone number stored in Airtable, formatted as a WhatsApp chatId (`phone@c.us`).

---

## 10. Airtable table rules

The project uses two tables:

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

For this phone-call workflow:

```text
Original Source = Form when the customer starts from the form
Last Contact Channel = Form when the form creates the row
Last Contact Channel = Outbound when the Telnyx AI call happens
```

---

## 11. Recommended DEV/PROD behavior

### DEV workflow

```text
Table = customers_dev
Environment = dev
Use test phone numbers
Use test credentials if possible
```

### PROD workflow

```text
Table = customers_prod
Environment = prod
Use real phone number
Use production credentials
```

---

## 12. GitHub and workflow versioning

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

## 13. Common mistakes to avoid

```text
Do not call customers without Opt-In.
Do not test with customers_prod.
Do not send real customers to customers_dev.
Do not store API keys directly in workflow JSON.
Do not delete Airtable fields before checking n8n mappings.
Do not rename Airtable fields without updating n8n nodes.
Do not change Original Source after the row was created.
Do update Last Contact Channel when a new contact happens.
```

---

## 14. Short summary

```text
Form collects basic customer data
Opt-In protects consent
AI cleans phone and first name
Airtable row is created
WhatsApp message warns customer about call
Telnyx AI Assistant starts outbound call
Call SID is saved
Telnyx post-call webhook receives transcript
AI extracts structured financing data
Airtable is updated
Final WhatsApp message confirms saved data
```

Most important rule:

```text
Initial form creates the lead.
AI phone call completes the qualification.
Transcript extraction updates the same Airtable row.
```