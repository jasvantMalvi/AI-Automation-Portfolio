# AI Outlook Auto Reply Agent

An automated Outlook email-processing workflow built with n8n, Microsoft Outlook, OpenAI GPT-5 Mini, a Structured Output Parser, Google Sheets, and a scheduled loop. It checks for unread emails every two minutes, retrieves up to 10 unread messages per fetch, classifies each message, generates a structured reply, records the email and proposed reply in Google Sheets, sends a new Outlook message to the original sender, and marks the original message as read.

> **Important:** This workflow sends AI-generated replies automatically. It does not ask a person to approve the reply before sending.

## Workflow Overview

The supplied export contains **18 nodes**: 9 functional nodes and 9 sticky-note annotations.

```text
Schedule Trigger (every 2 minutes)
              ↓
Get many messages (up to 10 unread)
              ↓
Loop Over Items (one message at a time)
              ↓
           AI Agent
          ↙        ↘
 OpenAI GPT-5 Mini  Structured Output Parser
              ↓
     Append row in Google Sheets
              ↓
       Send a message (Outlook)
              ↓
     Update a message (mark as read)
              └────────── back to Loop Over Items
```

The Split In Batches loop returns to the loop node after each processed email. The workflow continues until the items fetched in that execution have been processed.

## Main Workflow Stages

### 1. Schedule Trigger

`Schedule Trigger` starts the workflow every **2 minutes**.

### 2. Get unread Outlook messages

`Get many messages` uses the Microsoft Outlook node with:
- Operation: `getAll`
- Maximum messages per fetch: `10`
- Read status filter: `unread`
- Download attachments: enabled

The workflow fetches unread messages only. It does not contain a separate sender allowlist, subject filter, folder filter, or exclusion for newsletters and automated notifications.

### 3. Loop Over Items

`Loop Over Items` passes messages to the AI Agent one at a time. After processing, the original Outlook message is passed to `Update a message`, then the connection returns to the loop.

### 4. AI email classification and reply generation

`AI Agent` receives the email subject and `bodyPreview`. It uses **GPT-5 Mini** through `OpenAI Chat Model`.

The system prompt requires the model to classify each email into exactly one of these categories:

- Inquiry
- Complaint
- Meeting
- Support
- Sales
- Other

It must then generate:
- A concise, professional, polite reply.
- A short reply subject.
- A sender greeting only when the sender's name is clearly available.

The system prompt instructs the model not to invent facts, prices, policies, order details, dates, or promises. When information is missing, it should acknowledge the email and ask for the missing details.

`Structured Output Parser` requires this response structure:

| Field | Purpose |
|---|---|
| `category` | One of the six email categories |
| `subject` | Subject for the outgoing message |
| `reply` | Generated reply body |

### 5. Google Sheets logging

`Append row in sheet` appends one record to the `Sheet1` tab of the configured spreadsheet.

| Column | Value recorded |
|---|---|
| Date & Time | Current time formatted in `Asia/Kolkata` |
| Category | AI-generated email category |
| Sender's Email | Sender from the original Outlook message |
| Email Message | Original email subject and body preview |
| Email Reply | Generated reply subject and body |
| Status | `Replied` |

The operation is **append**, not update. Each processed email adds a new row.

### 6. Send the reply through Outlook

`Send a message` uses the sender from the original Outlook message as the recipient. It uses the subject and body returned by the AI Agent.

The node is configured as a send-message operation. It sends a new Outlook message; it is not configured as an Outlook reply-in-thread operation.

### 7. Mark the original email as read

`Update a message` updates the original message ID from `Loop Over Items` and sets `isRead` to `true`. This happens after the send node in the current connection path.

## Screenshots

See [screenshots/README.md](./screenshots/README.md) for the workflow overview, an Outlook inbox example, and the Google Sheets log example.

## Technology Stack

- **n8n** — scheduling, looping, and workflow orchestration
- **Microsoft Outlook** — retrieving unread emails, sending messages, and marking originals as read
- **OpenAI GPT-5 Mini** — email classification and reply drafting
- **Structured Output Parser** — consistent category, subject, and reply fields
- **Google Sheets** — email and reply log
- **JavaScript expressions** — dynamic field mapping and India time formatting

## Setup

1. Import `workflow.json` into your n8n instance. The exported workflow is inactive.
2. Reconnect Microsoft Outlook credentials to `Get many messages`, `Send a message`, and `Update a message`.
3. Reconnect OpenAI credentials to `OpenAI Chat Model`.
4. Create a Google Spreadsheet with a tab named `Sheet1`.
5. Add these exact headers: `Date & Time`, `Category`, `Sender's Email`, `Email Message`, `Email Reply`, and `Status`.
6. Replace `REPLACE_WITH_YOUR_GOOGLE_SHEET_ID` in `Append row in sheet` with your spreadsheet ID.
7. Reconnect Google Sheets credentials.
8. Test with a controlled Outlook mailbox and non-sensitive test emails from each category.
9. Check that the generated recipient, subject, body, sheet row, and read status are correct.
10. Keep the workflow inactive until sender exclusions, human review requirements, and failure handling have been considered.

## Important Limitations and Production Checks

### 1. AI replies are sent without human approval

The workflow sends the model's reply directly to the sender. There is no approval step or review queue. This can cause incorrect or inappropriate replies to be sent externally. For business use, add a human approval step, at least for complaints, sales commitments, billing, legal, security, and other sensitive messages.

### 2. The AI only receives the subject and body preview

The Outlook node downloads attachments, but the AI Agent prompt only passes `subject` and `bodyPreview`. No attachment extraction or attachment analysis node is present. Do not describe this workflow as reading attached files.

### 3. The email log is written before the send attempt

The Google Sheets row is appended with status `Replied` before `Send a message` executes. If Outlook sending fails, the sheet can still show `Replied` even though the message was not delivered. A more reliable design logs an initial `Pending` status and updates it to `Sent` only after successful send confirmation.

### 4. It sends a new message, not an Outlook thread reply

The Outlook node uses the send-message operation. Although the generated subject normally begins with `Re:`, the workflow does not use a dedicated reply operation or explicitly preserve the original conversation thread.

### 5. No sender or auto-reply exclusions

The only retrieval filter is unread status. The workflow does not exclude no-reply addresses, mailing lists, newsletters, internal automated alerts, or the user's own messages. Add filters before enabling unattended sending.

### 6. Duplicate replies are still possible after partial failures

The original email is marked as read only after the send node. If an email is sent successfully but the subsequent mark-as-read operation fails, that original message may remain unread and be fetched again on a later schedule. Add an idempotency check based on the Outlook message ID before sending.

### 7. The fetch is limited to 10 unread messages per execution

The workflow fetches at most 10 unread messages each time the schedule runs. It does not retrieve an unlimited mailbox backlog in one execution.

### 8. No dedicated error-handling route

There is no separate error branch for Outlook retrieval, AI parsing, Google Sheets append, message sending, or marking a message as read. Add retries and failure alerts before relying on it for a business inbox.

### 9. Email content is untrusted input

Incoming email content can contain misleading instructions or malicious prompt-injection text. The AI should treat message content as data, not instructions. Keep tool access disabled for this agent unless there is a clear need, and add safeguards for sensitive categories.

## Current Project Status

The supplied workflow automates unread-email retrieval, AI classification, reply generation, Google Sheets logging, Outlook message sending, and marking the original message as read. It is an **automatic reply prototype**, not a supervised help-desk system. Before business use, the highest-priority improvements are human approval, sender exclusions, correct delivery status logging, and duplicate protection.

## Author

<p align="left">
  <b>Jasvant Malvi</b><br>
  <a href="https://github.com/jasvantMalvi">GitHub</a> ·
  <a href="https://www.linkedin.com/in/jasvant-malvi-810305429/">LinkedIn</a>
</p>
