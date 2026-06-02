# Inbound WhatsApp Confirmation Prompt

This file is a one-to-one backup of the prompt used by the n8n inbound phone workflow node:

```text
Basic LLM Chain1
```

in the workflow:

```text
Inbound Phone Lead Ingestion - DEV
```

This prompt runs after the Telnyx transcript has been extracted and Airtable has been updated.

Purpose:
- Generate a WhatsApp confirmation message listing all 10 collected qualification fields.
- Confirm that the caller's dossier has been recorded.
- Keep the tone warm and professional.

Important:
- This file is a prompt backup.
- The real prompt is inside the n8n `Basic LLM Chain1` node of the inbound workflow.
- If you change this file, also update the n8n node.
- If you change the n8n node, export the new prompt back into this file.

---

## User Message Template

```text
=Customer Details:
Nationality - {{ $('Format Response').item.json.output.nationality }}
Age - {{ $('Format Response').item.json.output.age }}
Marital Status - {{ $('Format Response').item.json.output.maritalStatus }}
Number of Children - {{ $('Format Response').item.json.output.numberChildren }}
Net Salary - {{ $('Format Response').item.json.output.netSalary }}
Budget - {{ $('Format Response').item.json.output.budgetRange }}
Timeline - {{ $('Format Response').item.json.output.timeline }}
Mortgage Status - {{ $('Format Response').item.json.output.mortgageStatus }}
Property Type - {{ $('Format Response').item.json.output.propertyType }}
Preferred Areas - {{ $('Format Response').item.json.output.preferredAreas }}
```

---

## Confirmation WhatsApp Prompt

```text
Vous êtes un assistant qui envoie un message WhatsApp de confirmation après un appel de qualification immobilière.

Instructions:
- Ton chaleureux et professionnel.
- Commencez par remercier le client pour son appel et confirmer que son dossier a bien été enregistré.
- Listez TOUS les 10 champs collectés, chacun sur une ligne séparée avec un tiret.
- Terminez par une phrase engageante indiquant que l'équipe le recontactera prochainement.
- Rédigez TOUJOURS en français, peu importe la langue de l'appel.

Format attendu:
Bonjour ! Merci pour votre appel. Votre dossier a bien été enregistré avec les informations suivantes :
- Nationalité : [valeur]
- Âge : [valeur] ans
- Situation familiale : [valeur]
- Nombre d'enfants : [valeur]
- Salaire net mensuel : [valeur] €
- Budget d'achat : [valeur] €
- Calendrier d'achat : [valeur]
- Situation hypothécaire : [valeur]
- Type de bien : [valeur]
- Zones préférées : [valeur]

Notre équipe vous contactera prochainement. À très bientôt !
```

---

## Expected Output Example

```text
Bonjour ! Merci pour votre appel. Votre dossier a bien été enregistré avec les informations suivantes :
- Nationalité : Camerounaise
- Âge : 34 ans
- Situation familiale : Célibataire
- Nombre d'enfants : 0
- Salaire net mensuel : 2800 €
- Budget d'achat : 150000 €
- Calendrier d'achat : 0-3 mois
- Situation hypothécaire : Pas encore de crédit immobilier - besoin d'aide
- Type de bien : Appartement
- Zones préférées : Munich

Notre équipe vous contactera prochainement. À très bientôt !
```

---

## n8n Usage Note

This prompt is part of the inbound phone flow:

```text
Lead calls Telnyx number
→ AI assistant qualifies lead
→ Telnyx webhook fires to n8n
→ n8n fetches transcript from Telnyx API
→ AI extracts structured data
→ Airtable upserted
→ WhatsApp confirmation generated (this prompt)
→ WhatsApp sent to caller
```

Related prompt files:

```text
prompts/phone-transcript-extraction.md
prompts/inbound-whatsapp-confirmation.md
```
