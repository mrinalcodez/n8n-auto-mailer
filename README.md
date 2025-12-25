# AI-Powered Email & Calendar Automation using n8n and Google Gemini

This project is an automation system built with **n8n** that uses **Google Gemini (Generative AI)** to understand natural-language requests and perform intelligent actions such as scheduling calendar events, creating Google Meet meetings, fetching unread emails, or responding with a regular AI-generated reply.

The project consists of **two n8n workflows**:
1. A primary workflow that handles intent detection and orchestration
2. A secondary workflow that fetches and emails unread Gmail messages

---

## Features

- Natural language understanding using Google Gemini
- Automatic intent classification:
  - Schedule calendar events
  - Schedule meetings with Google Meet
  - Fetch unread emails
  - Handle regular (non-calendar, non-email) requests
- Google Calendar integration
- Gmail integration
- Fully containerized using Docker
- JSON-only AI responses for reliable automation

---

## Workflows Overview

### Workflow 1: `My workflow` (Primary Workflow)

Responsibilities:
- Triggered on a schedule
- Sends user input to Google Gemini
- Forces Gemini to return valid JSON only
- Parses and routes the response based on intent
- Creates Google Calendar events
- Creates Google Meet meetings when required
- Sends confirmation emails
- Triggers the secondary workflow to fetch emails when requested

Intent Types Handled:
- `schedule with meet`
- `schedule`
- `send mails`
- `regular`

---

### Workflow 2: `My workflow 2` (Email Fetch Workflow)

Responsibilities:
- Fetches unread emails from Gmail
- Formats email content as HTML
- Sends a consolidated unread email report via Gmail

---

## Prerequisites

Before installing, ensure the following are available:

- Docker installed
- Docker Compose (optional but recommended)
- A Google account
- Gmail API enabled
- Google Calendar API enabled
- Google AI Studio access for Gemini API key
- n8n Docker image

---

## Installation

### Step 1: Pull the Latest n8n Docker Image

```bash
docker pull n8nio/n8n:latest
```

### Step 2: Run n8n Using Docker
```
docker run -it --rm \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```

Once the container is running, open n8n in your browser:
```
http://localhost:5678
```

### Workflow Import

- Open the n8n UI

- Navigate to Workflows

- Click Import from File

- Import the workflows in the following order:

  1. My workflow 2

  2. My workflow
Note: Importing in this order is required because the primary workflow executes the secondary workflow internally.

## API and Credential Configuration

### Google Gemini API Key

1. Go to **Google AI Studio**  
   https://aistudio.google.com/

2. Generate a new API key

3. Open **My workflow** in n8n

4. Locate the **HTTP Request** node

5. Replace the placeholder `***` with your Gemini API key in the query parameters

**Example:**

```json
"key": "YOUR_GEMINI_API_KEY"
```

### Google OAuth Credentials

You must configure the following credentials inside **n8n**:

#### Gmail OAuth2

Used for:
- Sending confirmation emails
- Sending unread email summaries
- Fetching unread emails

#### Google Calendar OAuth2

Used for:
- Creating calendar events
- Creating meetings with Google Meet links

Ensure the following permissions (scopes) are enabled:
- Gmail read and send access
- Google Calendar read and write access

## Timezone and Date Handling

- All date and time calculations are done using:
`Asia/Kolkata (IST, +05:30)`
- If the user does not specify a date, the current date is used
- If only a start time is provided, the end time is automatically set to **1 hour later**
- All timestamps follow the **RFC3339** standard

---

## Gemini Response Contract

The Gemini model is strictly instructed to:

- Always return **valid JSON only**
- Never return Markdown, code blocks, or free-form text
- Follow a fixed schema depending on the detected intent

### Supported Response Types

#### Schedule a Meeting (with Google Meet)

```json
{
"type": "schedule with meet",
"title": "Team Meeting",
"description": "Weekly sync-up",
"start": "2025-01-10T10:00:00+05:30",
"end": "2025-01-10T11:00:00+05:30",
"reply": "Your meeting has been scheduled successfully."
}
```

Schedule a Regular Event
```json
{
  "type": "schedule",
  "title": "Workout",
  "description": "Morning exercise",
  "start": "2025-01-10T07:00:00+05:30",
  "end": "2025-01-10T08:00:00+05:30",
  "reply": "Your event has been added to the calendar."
}
```

Fetch Unread Emails
```
{
  "type": "send mails"
}
```

Regular AI Response
```
{
  "type": "regular",
  "reply": "Here is the information you requested."
}
```

Testing the Workflow

You can test the workflow by editing the Edit Fields node in My workflow.

Example: Schedule a Meeting
```
{
  "prompt": "Schedule a meeting tomorrow at 4 PM",
  "date": "2025-01-09T09:00:00+05:30"
}
```

Example: Fetch Unread Emails
```
{
  "prompt": "Show my unread emails",
  "date": "2025-01-09T09:00:00+05:30"
}
```

Email Output

- Confirmation emails are sent after successful event or meeting creation

- Unread emails are consolidated into a single HTML-formatted email

- Each email includes:

  1. Subject

  2. Sender address

  3. Recipient address

  4. Email body content
