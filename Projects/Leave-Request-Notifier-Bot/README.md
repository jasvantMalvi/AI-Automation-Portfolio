# Leave Request Notifier Bot

An employee leave-request notification workflow built with Google Forms, Google Sheets, n8n, OpenAI GPT-5 Mini, a Structured Output Parser, and Gmail. When a new leave-request row appears in the connected Google Sheet, the workflow extracts the employee and leave details, asks an AI Agent to prepare a neutral leave summary and inclusive date duration, updates the matching sheet record with the summary and `Pending` status, and sends an HTML email notification.

> **Important:** This workflow prepares and forwards a leave request. It does not approve or reject leave, and it does not implement a manager decision or approval workflow.

## Workflow Overview

The supplied export contains **11 nodes**: 7 functional nodes (including the AI model and output parser) and 4 sticky-note annotations.

```text
Google Sheets Trigger (new row; checks every minute)
                     ↓
       Extract Employee Details (Set)
                     ↓
       AI Leave Request Analyzer
          ↙                 ↘
 OpenAI Chat Model     Structured Output Parser
                     ↓
          Update Leave Record
                     ↓
         Notify Manager via Email
```

## Workflow Stages

### 1. New Leave Request Trigger

`01 – New Leave Request Trigger` is a Google Sheets Trigger configured for the `rowAdded` event. It polls the configured spreadsheet every minute and starts the workflow when a new response row is detected.

### 2. Extract Employee Details

`02 – Extract Employee Details` maps the incoming Google Sheets columns into standardized fields:

| Output field | Source sheet column |
|---|---|
| `name` | `Employee Name` |
| `email` | `Employee Email  ` |
| `department` | `Department  ` |
| `leave_type` | `Leave Type  ` |
| `start_date` | `Start Date` |
| `end_date` | `End Date` |
| `reason_for_leave` | `Reason for leave` |

**Column-name note:** The exported expressions contain trailing spaces in `Employee Email  `, `Department  `, and `Leave Type  `. This README and the JSON retain those existing names because the workflow uses them in its mappings.

### 3. AI Leave Request Analyzer

`03 – AI Leave Request Analyzer` uses **GPT-5 Mini** and the Structured Output Parser.

The prompt asks the model to:
- Identify employee details and leave type.
- Calculate leave duration, including both the start and end dates.
- Summarize the reason for leave.
- Highlight important observations.
- Set the leave status to `Pending`.
- Avoid approving or rejecting the request.

The parser example defines these output keys:

| Key | Example |
|---|---|
| `employee` | `Rahul Patel` |
| `leave_type` | `Sick Leave` |
| `duration` | `2 Days` |
| `summary` | `Employee requested two days of sick leave due to fever.` |
| `status` | `Pending` |

The date calculation and summary are AI-generated. The supplied workflow does not include a separate JavaScript date-validation step.

### 4. Update Leave Record

`04 – Update Leave Record` uses the Google Sheets `update` operation. It matches the row using the `Employee Email  ` column and writes:
- `AI Summary` from the model's summary.
- `Leave Status` from the model's status.
- `Employee Email  ` using an expression based on the model's `employee` output.

**The email mapping needs correction before production use.** The exported expression attempts to extract an email address from `output.employee` using a parenthesis-based regular expression. However, the Structured Output Parser example defines `employee` as a plain name (`Rahul Patel`), not a name-plus-email string. These two configurations do not agree. A safer mapping is to use the original extracted email directly, for example:

```javascript
{{ $('02 – Extract Employee Details').item.json.email }}
```

Use the exact node name as it appears in your n8n editor; the exported node name currently has a leading tab character before `02`.

### 5. Notify Manager via Email

`05 – Notify Manager via Email` sends an HTML-formatted notification with:
- Employee name and email.
- Department and leave type.
- Start date and end date.
- AI-generated leave duration.
- AI-generated leave summary.
- Current status displayed as `PENDING`.
- A reminder for the recipient to review the request and update the status after making a decision.

**Recipient mapping needs attention.** Despite the node name and message body saying the notification is for the manager, the exported `sendTo` expression uses the employee's own email from `02 – Extract Employee Details`. The supplied screenshot shows a delivered email with “Dear Manager,” but the actual recipient mapping is still the employee email. For a real manager notification, replace this expression with a configured manager/HR address or a manager email field from the form or employee directory.

## Google Sheets Structure

The screenshot shows these columns in the connected response sheet:

- `Timestamp`
- `Employee Name`
- `Employee Email`
- `Department`
- `Leave Type`
- `Start Date`
- `End Date`
- `Reason for leave`
- `AI Summary`
- `Leave Status`
- `Manager Remarks`

The response sheet contains a sample leave request with status `Pending`. The workflow updates the existing row rather than appending a separate processed-request row.

## Screenshots

See [screenshots/README.md](./screenshots/README.md) for the supplied form, workflow, Google Sheets, and email screenshots.

## Technology Stack

- Google Forms — employee leave request submission
- Google Sheets Trigger — detects new response rows
- Google Sheets — stores requests, AI summaries, and status
- n8n — workflow automation
- OpenAI GPT-5 Mini — leave request summarization
- Structured Output Parser — structured AI response
- Gmail — HTML email notification

## Setup

1. Import `workflow.json` into your n8n instance. The exported workflow is inactive.
2. Create or connect a Google Spreadsheet containing the response columns listed above.
3. Replace `REPLACE_WITH_YOUR_GOOGLE_SHEET_ID` in both Google Sheets nodes with your spreadsheet ID.
4. Reconnect Google Sheets Trigger and Google Sheets OAuth credentials.
5. Reconnect the OpenAI Chat Model credentials.
6. Reconnect Gmail credentials.
7. Correct the email matching expression in `04 – Update Leave Record` so it uses the original extracted employee email, not a parsed AI name field.
8. Decide who should receive the manager notification. Replace the Gmail recipient expression with the correct manager/HR address or a manager email field.
9. Submit a new test response through the leave form.
10. Confirm that the workflow updates the intended row, stores the AI summary, leaves the status as `Pending`, and sends the email to the intended manager address.
11. Test requests with one-day leave and multi-day leave to check inclusive duration handling.
12. Activate the workflow only after the mapping and recipient checks pass.

## Important Limitations and Checks

### 1. The workflow does not approve or reject leave

It always asks the AI to mark a request as `Pending`. There is no manager approval link, decision form, approval button, or status-change branch in this export. The manager must update the decision separately.

### 2. The email recipient does not match the manager notification wording

The Gmail node is named `Notify Manager via Email` and the message says “Dear Manager,” but the `sendTo` expression points to the employee email. Route this email to the manager/HR address before using it for real requests.

### 3. The Google Sheets matching-key expression is inconsistent with the parser schema

The update node attempts to extract an email address from `output.employee`, while the parser example contains only an employee name. Use the original email field from the Set node as the matching value, then test an update against the intended row.

### 4. Date duration is calculated by the AI

The prompt asks for inclusive date counting, but the workflow has no independent date arithmetic or validation node. Verify date formats and duration results, especially for requests that cross month or year boundaries.

### 5. Spreadsheet column headers contain trailing spaces in the export

Some expressions refer to `Employee Email  `, `Department  `, and `Leave Type  ` with trailing spaces. Renaming those headers without updating the node expressions and Google Sheets schema may break the workflow.

### 6. No leave balance, policy, or overlap validation

The workflow does not check remaining leave entitlement, public holidays, weekends, overlapping approved leave, or department staffing requirements.

### 7. No dedicated error-handling route

The supplied workflow has no explicit error branch for missing form data, AI parsing failure, Google Sheets update failure, or Gmail delivery failure.

### 8. Personal and employment information

Leave reasons and employee contact details can be sensitive. Limit spreadsheet and email access to authorized HR/management personnel, and avoid including unnecessary personal details in notifications.

## Current Project Status

The screenshots show a successful sample execution and an email notification with the request marked `Pending`. The exported workflow automates intake, AI summarization, sheet update, and email notification. It is **not a complete leave-management or approval system**. The recipient mapping and employee-email matching expression should be corrected and retested before production deployment.

## Author

**Jasvant Malvi**

- GitHub: https://github.com/jasvantMalvi
- LinkedIn: https://www.linkedin.com/in/jasvant-malvi-810305429/
