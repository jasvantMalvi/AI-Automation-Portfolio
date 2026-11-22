# AI Attendance Tracking Bot

A WhatsApp-based attendance recording workflow built with n8n and Google Sheets. It extracts the sender's details and attendance message, checks whether the same mobile number has already submitted an attendance record for the current date, then records the attendance or sends a duplicate-entry notification.

## Key Features

- Receives attendance messages through WhatsApp
- Extracts sender name, mobile number, attendance text, date, and time
- Converts the incoming WhatsApp timestamp to India time (`Asia/Kolkata`)
- Checks Google Sheets for an existing record using mobile number and date
- Saves a new attendance record when no matching record exists
- Sends a successful attendance confirmation through WhatsApp
- Sends an “Attendance Already Recorded” message when a matching record exists

## Workflow Overview

1. `WhatsApp Trigger` receives an incoming WhatsApp message.
2. `Code in JavaScript` extracts the sender's profile name, mobile number, message text, and WhatsApp timestamp. It converts the timestamp to `Asia/Kolkata` and formats the date as `DD/MM/YYYY` and time as `HH:MM:SS`.
3. `Get row(s) in sheet` searches the attendance sheet using two filters:
   - `date` equals the extracted date
   - `mobile` equals the sender's mobile number
4. `If` checks whether the lookup result is empty.
5. If no matching attendance row exists:
   - `Append row in sheet` saves the name, mobile number, date, time, and attendance text.
   - `Send message` sends a WhatsApp attendance confirmation.
6. If a matching record already exists:
   - `Send message1` informs the sender that attendance has already been recorded for that date.

## Technology Stack

- n8n
- WhatsApp Business Cloud API
- WhatsApp Trigger
- Google Sheets
- JavaScript (n8n Code node)
- `Intl` date/time formatting with `Asia/Kolkata`

## Google Sheets Structure

The workflow uses a single attendance sheet with these columns:

| Column | Purpose |
|---|---|
| `name` | WhatsApp profile name extracted from the sender's contact |
| `mobile` | Sender's WhatsApp mobile number |
| `date` | Attendance date in `DD/MM/YYYY` format |
| `time` | Attendance time in `HH:MM:SS` format |
| `attendance` | Text submitted by the user, such as `Present` |

The duplicate check uses the combination of `mobile` and `date`. The workflow does not create or search a separate Attendance ID.

## Workflow File

[Download n8n Workflow JSON](workflow.json) — Sanitized workflow export for importing into n8n.

## Screenshots

### 1. n8n Workflow Architecture
![Workflow Overview](screenshots/workflow-overview.png)

### 2. Google Sheets Attendance Records
![Attendance Google Sheet](screenshots/attendance-google-sheet.png)

### 3. WhatsApp Attendance Confirmation and Duplicate Alert
![WhatsApp Attendance Messages](screenshots/whatsapp-attendance-messages.png)

## Setup

1. Import `workflow.json` into your n8n instance. The exported workflow is inactive by default.
2. Create a Google Spreadsheet with the exact column headers `name`, `mobile`, `date`, `time`, and `attendance`.
3. Replace `REPLACE_WITH_YOUR_GOOGLE_SHEET_ID` in both Google Sheets nodes with your spreadsheet ID and select the correct worksheet.
4. Connect your own Google Sheets credentials.
5. Connect your own WhatsApp Business API credentials to the WhatsApp Trigger and both WhatsApp send-message nodes.
6. Replace `REPLACE_WITH_YOUR_WHATSAPP_PHONE_NUMBER_ID` in both send-message nodes with your own WhatsApp Business phone number ID.
7. Configure your WhatsApp webhook and ensure the trigger receives message events.
8. Test with one attendance message and verify that the record is appended with the correct date and time.
9. Send a second attendance message from the same mobile number on the same date and verify that the duplicate alert is sent.
10. Activate the workflow only after checking credentials, sheet mappings, date formatting, and duplicate behavior.

The export removes credential references, webhook IDs, the original spreadsheet ID, the original WhatsApp phone-number ID, and instance-specific metadata. Reconnect your accounts and configure your own values after import.

## Current Limitations and Recommended Improvements

The following points are visible in the supplied workflow export:

- **Attendance text is not validated:** The Code node reads the message text as submitted and the workflow appends it without checking whether it is an allowed value such as `Present` or `Absent`. Add validation before the Google Sheets lookup.
- **Duplicate detection uses mobile number and date, not an Attendance ID:** The workflow sticky note mentions a unique Attendance ID, but the actual Google Sheets lookup uses `date` and `mobile`. The spreadsheet shown also has no Attendance ID column.
- **No explicit empty-message or non-text handling:** The Code node reads `message.text.body`. Messages without a text body can result in an empty attendance value. Add a guard for unsupported message types and blank input.
- **Name comes from the WhatsApp profile:** The workflow uses `contacts[0].profile.name`. This is not an independently verified student identity. Consider mapping phone numbers to an approved student roster.
- **No authorization or roster validation:** The workflow does not check whether the incoming mobile number belongs to an authorized student before recording attendance.
- **Hard-coded WhatsApp phone-number ID:** The original send nodes use a configured WhatsApp Business phone-number ID. Replace it with your own account's value after importing.
- **Date/time uses India timezone:** The timestamp is converted using `Asia/Kolkata`. This is appropriate for an India-based attendance sheet, but it is not configurable per user or institution.

The screenshots demonstrate a successful attendance confirmation and a same-day duplicate alert. They do not prove that the workflow validates student identity or restricts attendance values.

## Repository Structure

```text
AI-Attendance-Tracking-Bot/
├── screenshots/
│   ├── README.md
│   ├── workflow-overview.png
│   ├── attendance-google-sheet.png
│   └── whatsapp-attendance-messages.png
├── README.md
└── workflow.json
```
