# Daily Motivation Message Sender Bot

An n8n prototype that generates short, practical motivational messages with an AI Agent, sends the message to a configured Telegram chat, and appends a record to Google Sheets.

> **Important:** The workflow export currently schedules execution every 2 minutes, despite the project name and the daily-motivation use case. Do not activate it until the schedule is changed to once per day and tested. Otherwise it can send repeated messages throughout the day.

## Workflow Overview

The supplied export contains **8 nodes**:
- 6 connected nodes in the active execution path.
- 1 disconnected Telegram Trigger.
- 1 disconnected OpenAI Chat Model.

The connected path is:

```text
Schedule Trigger (every 2 minutes in supplied export)
                 ↓
              AI Agent
            ↙         ↘
Send a text message   Code in JavaScript
    (Telegram)                ↓
                      Append row in sheet
                       (Google Sheets)
```

`Google Gemini Chat Model` is connected to the AI Agent as its language model. `OpenAI Chat Model` is present in the workflow but is not connected.

## Workflow Stages

### 1. Schedule Trigger

`Schedule Trigger` starts the workflow every **2 minutes** in the supplied JSON. This does not match a once-daily reminder schedule. Change the trigger to the desired daily time before enabling the workflow.

The export does not set an explicit timezone. Confirm the n8n workflow/instance timezone so the message is sent at the intended local time.

### 2. AI Agent and message generation

`AI Agent` uses the connected `Google Gemini Chat Model` and the system instructions included in the export.

The system instructions ask for a concise and practical motivational message focused on:
- Discipline and consistency
- Learning and growth
- Productivity and focus
- Responsibility and time management
- Self-improvement and long-term progress

They specify:
- Preferred length: 30–50 words
- Maximum length: 70 words
- Maximum 2–3 emojis
- One clear takeaway and an immediate action
- An action challenge beginning with `Today's Action:`
- A warm, professional, uplifting tone
- No generic clichés, toxic positivity, unrealistic promises, religious or political opinions

**Prompt correction in the public export:** the supplied AI Agent user-message field contained only the numeric value `6040153334`, not a useful instruction for generating a message. The public workflow replaces that invalid value with a concise motivation-generation instruction. The system prompt is otherwise retained.

### 3. Send a Telegram message

`Send a text message` sends the AI Agent output to Telegram.

In the original export, the Telegram `chatId` is hard-coded to one numeric ID. The public JSON replaces it with `REPLACE_WITH_YOUR_TELEGRAM_CHAT_ID`. This design sends to one configured destination; it is not a multi-user broadcast system.

### 4. Prepare the log record

`Code in JavaScript` reads the AI Agent output and creates these fields:
- `date` — formatted using `Asia/Kolkata`
- `time` — formatted using `Asia/Kolkata` in 24-hour format
- `motivation_message` — generated message text
- `created_at` — ISO timestamp
- `message_length` — message character count

### 5. Save the message in Google Sheets

`Append row in sheet` appends the generated message record to the configured spreadsheet's `Sheet1` tab.

The screenshot shows these columns:

| Column | Purpose |
|---|---|
| `date` | Local date in Asia/Kolkata |
| `time` | Local time in Asia/Kolkata |
| `motivation_message` | Full generated message |
| `created_at` | ISO timestamp |
| `message_length` | Character count |

The operation is **append**, so each execution creates a new row. There is no deduplication or once-per-day guard in the supplied path.

## Disconnected Nodes in the Export

Two nodes are present but disconnected:
- `Telegram Trigger` — does not start this scheduled workflow and does not receive user messages in the supplied connections.
- `OpenAI Chat Model` — is not used by the AI Agent; the connected model is Google Gemini.

The published workflow should either remove these unused nodes or connect them intentionally in a separate, clearly defined flow.

## Screenshots

See [screenshots/README.md](./screenshots/README.md) for the n8n workflow overview, sample Sheets log, and Telegram message examples.

## Technology Stack

- **n8n** — scheduled automation and workflow orchestration
- **Google Gemini Chat Model** — motivational message generation through the AI Agent
- **Telegram Bot API** — delivery of the generated message
- **Google Sheets** — message history and execution log
- **JavaScript** — timestamp formatting and log-field preparation

## Setup

1. Import `workflow.json` into n8n. The public export is inactive.
2. Change `Schedule Trigger` from every 2 minutes to once per day at your chosen send time.
3. Set and verify the workflow timezone (for example, `Asia/Kolkata`) to match the intended schedule.
4. Reconnect Google Gemini credentials for `Google Gemini Chat Model`.
5. Reconnect Telegram credentials for `Send a text message`.
6. Replace `REPLACE_WITH_YOUR_TELEGRAM_CHAT_ID` with the intended recipient chat ID.
7. Create or select a Google Spreadsheet with a tab named `Sheet1`.
8. Add these exact headers: `date`, `time`, `motivation_message`, `created_at`, `message_length`.
9. Replace `REPLACE_WITH_YOUR_GOOGLE_SHEET_ID` in `Append row in sheet`.
10. Reconnect Google Sheets credentials.
11. Execute a manual test and verify the Telegram message and appended row.
12. Confirm the scheduled trigger runs only once at the intended daily time before activating.

## Important Limitations and Production Checks

### 1. The current schedule is every 2 minutes

This is the highest-priority issue. As exported, the workflow can send a new motivation message every 2 minutes. Change it to the intended daily schedule before activation.

### 2. Telegram recipient is fixed

The supplied export uses a single hard-coded chat ID. It is suitable for one configured recipient only. It does not look up a list of recipients or send personalized messages to multiple users.

### 3. No once-per-day or duplicate guard

The current execution path has no check against the current date or an execution log before sending. If the schedule fires more than once, every run can generate and send another message.

### 4. Two nodes are unused

The Telegram Trigger and OpenAI Chat Model are disconnected. They do not participate in the execution path. The actual connected model is Google Gemini.

### 5. Logging and sending are parallel branches

The AI Agent has two outgoing connections: one to Telegram and one to the JavaScript logging branch. The sheet append is not downstream of successful Telegram delivery. Therefore, a row may be logged even if the Telegram send fails. Add delivery-status logging and error handling if the log must represent confirmed delivery.

### 6. No dedicated error handling

There is no explicit error route for AI generation, Telegram delivery, or Google Sheets append. Add retries or failure alerts if this is used as a recurring service.

### 7. No preference or opt-out controls

The workflow has one configured Telegram destination and no subscription, pause, quiet-hours, or unsubscribe handling. For a multi-user version, add recipient management and opt-out controls.

## Current Project Status

The supplied screenshots show motivational messages being sent to Telegram and message history being stored in Google Sheets. The supplied JSON, however, is configured to run every 2 minutes and uses a fixed Telegram chat ID. Treat this as a prototype until the schedule is changed, the destination is configured safely, and delivery/logging behaviour is tested.

## Author

<p align="left">
  <b>Jasvant Malvi</b><br>
  <a href="https://github.com/jasvantMalvi">GitHub</a> ·
  <a href="https://www.linkedin.com/in/jasvant-malvi-810305429/">LinkedIn</a>
</p>
