# Phone Transcript Extraction Prompt

This file is a one-to-one backup of the prompt used by the n8n phone-call transcript extraction step.

It is used in the `Format Response` node in two workflows:

```text
Data Ingestion Kameni - Finanzierung - DEV   (outbound phone workflow)
Inbound Phone Lead Ingestion - DEV           (inbound phone workflow)
```

The extraction prompt is identical in both workflows.
The user message template differs between them (see below).

Important:
- This file is a prompt backup.
- The real prompt is currently used inside the n8n `Format Response` node.
- If you change this file, also update the n8n node.
- If you change the n8n node, export the new prompt back into this file.

---

## User Message Template — Outbound workflow

```text
=Transcript Summary: {{ $json.body.data.transcript.map(t => t.role + ": " + t.message).join("\n")  }}
```

## User Message Template — Inbound workflow

The inbound workflow builds the transcript using a Code node ("Build Transcript") that
fetches messages from the Telnyx conversations API and filters out tool-call noise.
The Format Response node receives the clean transcript as:

```text
={{ $json.transcript }}
```

Where `$json.transcript` is a newline-separated string formatted as:
```text
Agent: <agent message>
Caller: <caller message>
...
```

---

## Extraction Prompt

```text
You are a smart assistant that extracts structured answers from a phone conversation transcript.

Focus on returning **only the buyer's answers**, not the questions themselves.

Extract these 10 fields with the SPECIFIED format:

1. **nationality** (string): The customer's nationality (e.g. "Cameroonian", "German", "French")
2. **age** (NUMBER, integer): The customer's age. Convert text to number (e.g. "forty" -> 40)
3. **maritalStatus** (string, MUST match one of): "Single", "Married", "Divorced", "Widowed", "Unknown"
4. **numberChildren** (NUMBER, integer): Number of children (0 if none mentioned)
5. **netSalary** (NUMBER, integer in EUR): Net monthly salary as a number, no currency symbol (e.g. "3000 euros" -> 3000)
6. **budgetRange** (NUMBER, integer in EUR): Property budget as a number (e.g. "around 400k" -> 400000)
7. **timeline** (string, MUST match one of): "0-3 months", "3-6 months", "6-12 months", "12+ months", "Just exploring"
8. **mortgageStatus** (string, MUST match one of): "No mortgage yet - needs help", "Mortgage in principle", "Mortgage approved", "Already has mortgage", "Unknown"
9. **propertyType** (string, MUST match one of): "Apartment", "House", "Detached house", "Semi-detached house", "Plot of land", "Commercial", "Other"
10. **preferredAreas** (string): Free text list of areas/neighbourhoods mentioned

Also include:
- **qualifiedLead** (boolean): true if customer has clear timeline (within 12 months), reasonable budget/income, and clarity on mortgage status; otherwise false

IMPORTANT:
- For singleSelect fields, you MUST use one of the exact values listed.
- For NUMBER fields, return integers without quotes, no currency symbols.
- If a field cannot be determined from the transcript, use "Unknown" for selects, 0 for numbers, empty string for text.
```

---

## Structured Output Parser Schema

```json
{
  "nationality": "Cameroonian",
  "age": 35,
  "maritalStatus": "Single | Married | Divorced | Widowed | Unknown",
  "numberChildren": 2,
  "netSalary": 3000,
  "budgetRange": 350000,
  "timeline": "0-3 months | 3-6 months | 6-12 months | 12+ months | Just exploring",
  "mortgageStatus": "No mortgage yet - needs help | Mortgage in principle | Mortgage approved | Already has mortgage | Unknown",
  "propertyType": "Apartment | House | Detached house | Semi-detached house | Plot of land | Commercial | Other",
  "preferredAreas": "Headington, Cowley, central Oxford",
  "qualifiedLead": true
}
```

---

## Expected Output Meaning

The AI should extract only the customer's answers from the phone transcript.

It should return clean structured data for Airtable:

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
- Select values must match Airtable exactly.
- Number values must be integers.
- Unknown select values should be `Unknown`.
- Missing number values should be `0`.
- Missing text values should be an empty string.
- Values saved must always be in English, regardless of the language spoken in the call
  (e.g. "Célibataire" → "Single", "Appartement" → "Apartment").
