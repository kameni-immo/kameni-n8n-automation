# Airtable Fields – Lead Automation Project

This file explains the Airtable tables and fields used by the Kameni lead automation project.

The project uses two customer tables:

```text
customers_dev  = test/development leads
customers_prod = real/live production leads
```

Both tables should have the same field names and same field options.  
This makes it easy to test a workflow in DEV and later copy the same logic to PROD.

---

## 1. Table purpose

| Table | Used by | Meaning |
|---|---|---|
| `customers_dev` | DEV workflows | Test data only |
| `customers_prod` | PROD workflows | Real customer data |

Simple rule:

```text
DEV workflow  → customers_dev  → Environment = dev
PROD workflow → customers_prod → Environment = prod
```

---

## 2. Customer identity fields

| Field | Type | Meaning | Example |
|---|---|---|---|
| `Name` | Text | Full customer name | Maria Becker |
| `First Name` | Text | First name used in messages | Maria |
| `Email` | Email | Customer email address | maria.becker@example.com |
| `Phone` | Phone | Customer phone number | +4915123456789 |

Notes:

```text
Name       = full legal/customer name
First Name = friendly name used in WhatsApp/SMS
Phone      = main matching field to find existing customers
```

---

## 3. Environment and source tracking

| Field | Type | Meaning | Example |
|---|---|---|---|
| `Environment` | Single select | Shows whether the row is test or live data | dev / prod |
| `Original Source` | Single select | First channel where the lead came from | WhatsApp |
| `Last Contact Channel` | Single select | Latest channel used with the lead | Outbound |

Important rule:

```text
Original Source should be set once when the customer is created.
Last Contact Channel should be updated every time there is a new contact.
```

Example:

```text
First contact:  Inbound Call
Original Source: Inbound Call
Last Contact Channel: Inbound Call

Second contact: WhatsApp
Original Source: Inbound Call
Last Contact Channel: WhatsApp

Third contact: Outbound Call
Original Source: Inbound Call
Last Contact Channel: Outbound
```

Recommended select values:

```text
Form
Inbound Call
WhatsApp
Outbound
```

---

## 4. Lead status fields

| Field | Type | Meaning | Example |
|---|---|---|---|
| `Pipeline Stage` | Single select | Current step of the lead | Awaiting Documents |
| `Opt In` | Checkbox | Customer agreed to be contacted | true |
| `Qualified Lead` | Checkbox | Lead is good enough to continue | true |

Recommended `Pipeline Stage` values:

```text
New
Qualifying
Awaiting Documents
Documents Validated
Booked
Lost
```

Simple meaning:

```text
New                 = lead was created
Qualifying          = AI/team is collecting missing info
Awaiting Documents  = customer must send documents
Documents Validated = documents were checked
Booked              = appointment is booked
Lost                = lead will not continue
```

---

## 5. Qualification fields

| Field | Type | Meaning | Example |
|---|---|---|---|
| `Nationality` | Text | Customer nationality | German |
| `Age` | Number | Customer age in years | 35 |
| `Marital Status` | Single select | Customer family status | Married |
| `Number of Children` | Number | Number of children | 2 |
| `Net Salary` | Currency | Monthly net salary | 3200 |
| `Budget` | Currency | Property purchase budget | 280000 |
| `Timeline` | Single select | When customer wants to buy | 6-12 months |
| `Mortgage Status` | Single select | Current financing status | No mortgage yet - needs help |
| `Property Type` | Single select | Desired property type | Apartment |
| `Preferred Areas` | Long text | Desired city/area | Frankfurt, Offenbach |

Recommended `Marital Status` values:

```text
Single
Married
Divorced
Widowed
Unknown
```

Recommended `Timeline` values:

```text
0-3 months
3-6 months
6-12 months
12+ months
Just exploring
```

Recommended `Mortgage Status` values:

```text
No mortgage yet - needs help
Mortgage in principle
Mortgage approved
Already has mortgage
Unknown
```

Recommended `Property Type` values:

```text
Apartment
House
Detached house
Semi-detached house
Plot of land
Commercial
Other
```

---

## 6. Phone call / Telnyx fields

| Field | Type | Meaning | Example |
|---|---|---|---|
| `Conversation ID` | Text | Telnyx conversation ID | conv_test_123 |
| `Call SID` | Text | Telnyx call SID | CA123456789 |
| `Call Status` | Single select | Current call status | Completed |

Recommended `Call Status` values:

```text
Not Initiated
Initiated
Answered
Completed
Failed
No Answer
```

Simple meaning:

```text
Conversation ID = ID from Telnyx conversation
Call SID        = ID from Telnyx call
Call Status     = what happened with the call
```

---

## 7. Document and financing process fields

| Field | Type | Meaning | Example |
|---|---|---|---|
| `Vorgangsnummer` | Text | Internal financing case number | KAM-2026-0001 |
| `ID Number` | Text | Customer ID/passport number | X1234567 |
| `Selbstauskunft Submitted` | Checkbox | Customer submitted the self-disclosure form | true |
| `Documents Folder URL` | URL | Link to Google Drive/Dropbox folder | https://example.com/folder |
| `Documents Complete` | Checkbox | All required documents are complete | false |
| `Calendly Booking` | Date/Time | Booked appointment date/time | 2026-05-20 10:00 |

Simple meaning:

```text
Selbstauskunft Submitted = form received
Documents Folder URL     = where documents are stored
Documents Complete       = documents are uploaded and checked
Calendly Booking         = appointment booked
```

---

## 8. How workflows should use the tables

### WhatsApp DEV workflow

```text
Table: customers_dev
Environment: dev
Original Source: WhatsApp only when creating new customer
Last Contact Channel: WhatsApp whenever customer sends a WhatsApp message
```

### WhatsApp PROD workflow

```text
Table: customers_prod
Environment: prod
Original Source: WhatsApp only when creating new customer
Last Contact Channel: WhatsApp whenever customer sends a WhatsApp message
```

### Phone / outbound DEV workflow

```text
Table: customers_dev
Environment: dev
Original Source: Form or Inbound Call when creating new customer
Last Contact Channel: Outbound when the Telnyx AI calls the customer
```

### Phone / outbound PROD workflow

```text
Table: customers_prod
Environment: prod
Original Source: Form or Inbound Call when creating new customer
Last Contact Channel: Outbound when the Telnyx AI calls the customer
```

---

## 9. Important automation rules

```text
1. DEV workflows must write only to customers_dev.
2. PROD workflows must write only to customers_prod.
3. Keep field names the same in both tables.
4. Do not rename fields without updating n8n workflows.
5. Do not delete fields before checking if n8n still uses them.
6. Original Source should not change after the row is created.
7. Last Contact Channel should change after every new interaction.
8. Phone should be used to find/update existing customers.
```

---

## 10. Example customer row

This is only fake example data.

| Field | Example |
|---|---|
| Name | Maria Becker |
| First Name | Maria |
| Email | maria.becker@example.com |
| Phone | +4915123456789 |
| Environment | dev |
| Original Source | WhatsApp |
| Last Contact Channel | WhatsApp |
| Pipeline Stage | Awaiting Documents |
| Qualified Lead | true |
| Nationality | German |
| Age | 35 |
| Marital Status | Married |
| Number of Children | 2 |
| Net Salary | 3200 |
| Budget | 280000 |
| Timeline | 6-12 months |
| Mortgage Status | No mortgage yet - needs help |
| Property Type | Apartment |
| Preferred Areas | Frankfurt, Offenbach |

---

## 11. Short summary

```text
customers_dev  = test table
customers_prod = live table

Environment = dev or prod
Original Source = first channel
Last Contact Channel = latest channel
Pipeline Stage = current process step
Qualified Lead = good lead or not
```
