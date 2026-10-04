# AI Speak-to-Lead System

This project is a starter backend for a lead-calling automation that matches the workflow in the supplied brief:

- Meta lead ads trigger a webhook
- the lead is sanitized and normalized
- the system checks for duplicates
- the system calls the lead within the allowed calling window
- the AI voice agent asks a few questions and tries to book a site visit
- WhatsApp follow-up is queued outside calling hours

## Stack

- Node.js + Express
- SnapServe outbound voice API
- WhatsApp Cloud API for SMS-style follow-up messages
- Google Sheets as the lead tracker for production workflows
- n8n workflow blueprint for orchestration
- JSON tracker for local testing and offline development

## Quick start

1. Install dependencies:
   npm install
2. Copy the environment settings:
   copy .env.example .env
3. Fill in your values in `.env`.
4. Start the service:
   npm start
5. Test the health endpoint:
   GET /health

## Production integrations

### Google Sheets

Set these environment values to store each new lead in a Google Sheet:

- `GOOGLE_SERVICE_ACCOUNT_EMAIL`
- `GOOGLE_PRIVATE_KEY`
- `GOOGLE_SHEET_ID`
- `GOOGLE_SHEET_RANGE`

The app appends rows in the format: lead_id, lead_name, lead_phone, lead_email, project_name, configuration, budget, timeline, source, location, language_style, status, created_at, last_attempt_at, updated_at.

### WhatsApp Cloud API

Set these environment values to send WhatsApp follow-up messages:

- `WHATSAPP_API_TOKEN`
- `WHATSAPP_PHONE_NUMBER_ID`
- `WHATSAPP_TEMPLATE_NAME` (optional if you want a plain text API call)

You can validate this with:

```bash
curl -X POST http://localhost:3000/api/whatsapp/test \
  -H "Content-Type: application/json" \
  -d '{"to":"+919876543210","message":"Hi Anita, this is a test message from your AI lead system."}'
```

## API

### POST /api/meta-lead

Sends a new inbound lead to the system.

Example payload:

```json
{
  "lead_name": "Anita",
  "lead_phone": "+91 98765 43210",
  "project_name": "Sky Villas",
  "configuration": "3 BHK",
  "budget": "1.2 to 1.5 Cr",
  "timeline": "1 to 3 months",
  "source": "Instagram",
  "location": "OMR, Chennai",
  "language_style": "English"
}
```

Response:

```json
{
  "status": "calling",
  "lead": { "lead_phone": "+919876543210" },
  "snapserve": { "status": "pending" }
}
```

### POST /api/test-call

Makes a real SnapServe trial call to the number in `TEST_TO_NUMBER`.

## Voice assistant prompt

The conversational guardrails are stored in `src/prompts.js` and cover:

- AI identification
- warm but short phone script
- visit booking flow
- no price or discount promises
- quiet-hour and DNC-safe call logic

## n8n workflow

The workflow template is in `n8n/ai-lead-voice-agent.json`.
It mirrors the structure referenced in the brief:

1. New Lead trigger
2. Clean Lead field mapper
3. Duplicate check
4. Save lead
5. Calling Hours check
6. WhatsApp message
7. Call Lead HTTP request
8. Mark Calling update

## Notes

- The app intentionally keeps external calls disabled unless `SNAPSERVE_API_KEY` and `SNAPSERVE_AGENT_ID` are configured.
- The service stores raw lead records in `data/leads.json` so you can test without a Google Sheet.
- To move this into production, replace the JSON file with Google Sheets or a real CRM connector and add the WhatsApp Cloud API call.
