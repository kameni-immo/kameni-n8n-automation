# Phone Contact Cleanup Prompt

This file is a one-to-one backup of the prompt used by the n8n phone workflow node:

```text
Basic LLM Chain
```

This prompt is used before the AI phone call starts.

Purpose:
- Clean and format the customer's phone number.
- Convert the phone number into E.164 format.
- Extract the customer's first name.
- Keep the customer email address available for Airtable.

Important:
- This file is a prompt backup.
- The real prompt is currently used inside the n8n `Basic LLM Chain` node.
- If you change this file, also update the n8n node.
- If you change the n8n node, export the new prompt back into this file.

---

## User Message Template

```text
=Customer Phone Number:  {{ $json['Phone Number'] }}
Customer First Name: {{ $json['Customer Name'] }}
Customer Email Address: {{ $json['Email'] }}
```

---

## Cleanup Prompt

```text
=You are a smart assistant that processes customer contact information to prepare for automated dialling and personalisation.

Your two tasks are:
### 1. Phone number Formatting

Take any phone number as entered by the customer and convert it to the standard E.164 international format.

Rules:

- Detect the country from the number's structure or prefix.
- Return only the E.164 format: `+[CountryCode][NationalNumber]`, with no spaces, dashes, or brackets.
- Clean the input first by removing all non-digit characters except a leading `+`.
- If the number:

  - Starts with `+` → clean and validate it.
  - Starts with `00` → convert to `+`.
  - Starts with a national prefix (e.g. `0`) → remove and replace with the correct country code based on number structure.
- If the number is incomplete or the country cannot be confidently determined, return: `Invalid number`.

Examples:

- `415 555 2671` → `+14155552671` (United States)
- `06 12 34 56 78` → `+33612345678` (France)
- `0049 151 23456789` → `+4915123456789` (Germany)
- `0151 23456789` → `+4915123456789` (Germany)
- `+61 412 345 678` → `+61412345678` (Australia)

### 2. First Name Extraction

Extract the customer's **first name** from their full name input.

Guidelines:

- Support a wide range of ethnically diverse names.
- The first name is usually the first word unless the input uses reversed formats (e.g. `Kim, Hana`). Handle these gracefully.
- Remove any extra titles (e.g. Mr, Mrs, Dr) or honorifics.
- Return only the first name, capitalized properly.
```

---

## Structured Output Parser Schema

```json
{
	"Customer Phone Number": "The customer phone number in E.164 format",
	"Customer First name": "The first name of the customer",
    "Customer Email Address": "The customer email address"
}
```

---

## Expected Output Meaning

The AI should return:

```text
Customer Phone Number = cleaned phone number in E.164 format
Customer First name = first name only
Customer Email Address = customer email address
```

Example:

```text
Input phone: 0049 151 23456789
Output phone: +4915123456789

Input name: Maria Becker
Output first name: Maria
```

---

## n8n Usage Note

This prompt supports the first part of the phone workflow:

```text
Form submission
→ clean contact data
→ create Airtable customer
→ send SMS
→ start AI phone call
```

The transcript extraction after the call is stored separately in:

```text
prompts/phone-transcript-extraction.md
```
