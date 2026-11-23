# AI Maintenance Dispatch System

A four-workflow maintenance request processing prototype built with n8n, JavaScript, webhooks, and Google Sheets. The package contains separate endpoints for request intake, issue analysis, technician matching, and status validation.

**Important:** The four workflows are separate webhook-triggered workflows. The supplied exports do not show an automated call chain between them. Each endpoint must currently be invoked separately by a caller or by an additional orchestration workflow. The package is a prototype, not a complete production dispatch platform.

## What the current package actually does

| Workflow | Current behaviour | Persistence |
|---|---|---|
| WF01 — Request Intake | Validates required employee/request fields, applies rule-based category and priority labels, creates a ticket ID, and returns a confirmation | Appends a ticket to the `Maintenance_Requests` Google Sheets tab |
| WF02 — Issue Analyzer | Builds an AI analysis prompt and parses a response; the active route currently returns a fixed mock HVAC result | No ticket update or sheet write |
| WF03 — Technician Matching | Reads technician records, matches by exact skill, availability, and workload, then returns a proposed assignment | Reads the technician sheet only; does not update the maintenance ticket |
| WF04 — Status Validation | Checks whether a supplied status belongs to an allowed list and returns a validation response | Does not update a ticket or sheet |

## Workflow Architecture

### WF01 — Maintenance Request Intake

**Webhook path:** `POST /maintenance-request`

Receives a JSON request and validates these required fields:

- `employee_name`
- `employee_email`
- `issue_description`
- `location`

The email is checked with a basic format expression. The workflow truncates the issue description to 3,000 characters, applies rule-based category and priority detection, creates a ticket ID, and appends the ticket to Google Sheets.

**Rule-based classification in the supplied code:**

- Categories: HVAC, Plumbing, Electrical, IT Equipment, or General Maintenance
- Critical keywords include fire, smoke, gas leak, electrocution, exposed live wire, and collapse
- Critical issues are saved with `status = Escalated` and `requires_human_review = true`
- Other urgent patterns can produce High priority; remaining requests default to Medium

This is keyword-based classification, not an AI analysis call.

### WF02 — Maintenance Issue Analyzer

**Webhook path:** `POST /maintenance-analyze`

Accepts `issue_description` and an optional `ticket_id`. The Prepare Prompt node builds an instruction for a maintenance issue analysis containing:

- `category`
- `priority`
- `summary`
- `recommended_action`
- `requires_human_review`

The parser restricts category and priority values to defined lists and forces human review when the parsed priority is Critical.

**Current implementation warning:** The active route is connected to `Mock AI Response`, which returns the same hard-coded HVAC / High example regardless of the submitted issue. The `Gemini Analyze` HTTP Request node is disconnected. Therefore, the current active path is not using live Gemini analysis.

The invalid-issue output of `Valid Issue Description?` is also not connected to a response node, so an empty issue may fail to receive a proper webhook response.

### WF03 — Technician Matching

**Webhook path:** `POST /maintenance-dispatch`

Accepts:

- `ticket_id`
- `category`
- `priority` (defaults to Medium)

The workflow reads technician records from Google Sheets. It selects technicians whose:

- `Skills` column contains an exact category match after comma-separated splitting
- `Availability` is `Available`
- `Active_Jobs` is lower than `Max_Jobs`

Eligible technicians are sorted by current active job count, and the lowest-load match is returned.

The response uses `dispatch_status = Awaiting Confirmation`. Critical requests are returned for manual review, and unmatched requests return a manual-assignment message.

**Current limitation:** The match is only returned by the webhook. This workflow does not update `assigned_technician_id` or another field in the original maintenance ticket, and it does not notify the technician.

### WF04 — Status Validation

**Webhook path:** `POST /maintenance-status`

Accepts `ticket_id`, `status`, and optional `resolution_notes`. The allowed status values in the supplied code are:

- Pending
- Analyzing
- Awaiting Approval
- Assigned
- In Progress
- On Hold
- Completed
- Rejected
- Escalated

For valid input, it returns the ticket ID, status, resolution notes, current UTC timestamp, and a success message.

**Current limitation:** This is validation only. No Google Sheets lookup or update is performed, so the maintenance ticket's stored status is not changed. The webhook response node is configured with HTTP response code 200, including for validation failures represented as `success: false`.

## Data Structure

### Maintenance_Requests

WF01 writes these fields:

| Field | Purpose |
|---|---|
| `ticket_id` | Generated maintenance ticket ID |
| `employee_name` | Requester's name |
| `employee_email` | Requester's email |
| `phone` | Optional phone number |
| `issue_description` | Maintenance issue, limited to 3,000 characters |
| `location` | Reported location |
| `category` | Rule-based issue category |
| `priority` | Rule-based priority |
| `status` | Pending or Escalated at intake |
| `assigned_technician_id` | Blank at intake |
| `created_at` | Creation timestamp |
| `updated_at` | Initial update timestamp |
| `resolution_notes` | Blank at intake |
| `requires_human_review` | Critical-issue review flag |

### Technician records

WF03 expects these technician columns:

- `Technician_ID`
- `Technician_Name`
- `Email`
- `Skills`
- `Availability`
- `Active_Jobs`
- `Max_Jobs`

The export uses a placeholder Google Sheet reference. Configure your own spreadsheet and the correct technician worksheet before execution.

## Technology Stack

- n8n — four webhook-based workflows
- JavaScript — request validation, rule-based classification, parsing, and technician matching
- Google Sheets — maintenance request and technician data
- Webhooks / JSON — workflow input and API responses
- Gemini HTTP Request node — present but disconnected in the current WF02 export
- Mock AI response — active WF02 analysis path

## Project Files

### n8n workflow exports

1. [WF01 — Maintenance Request Intake](workflows/WF01-Maintenance-Request-Intake.json)
2. [WF02 — AI Issue Analyzer](workflows/WF02-AI-Issue-Analyzer.json)
3. [WF03 — Technician Matching](workflows/WF03-Technician-Matching.json)
4. [WF04 — Status Validation](workflows/WF04-Status-Validation.json)

### Screenshots

See the [screenshots folder](screenshots/README.md) for the four workflow diagrams.

## Setup

1. Import the four JSON files into n8n individually.
2. Configure a Google Spreadsheet with a `Maintenance_Requests` tab and the ticket columns above.
3. Configure a technician worksheet with the technician fields above.
4. Replace each Google Sheets placeholder with your own spreadsheet and worksheet references.
5. Configure Google Sheets credentials in WF01 and WF03.
6. Configure each webhook URL and test the four endpoints independently.
7. Use test records first. Confirm that WF01 writes the request and WF03 can read the technician records.
8. Do not treat a WF03 match as an actual assignment until a separate update step persists the technician ID to the ticket.
9. Do not treat a WF04 success response as a saved status change; the current workflow does not persist status.

## Known Gaps — Fix Before Production

1. **No workflow orchestration:** WF01 does not call WF02, WF03, or WF04. A separate orchestrator or explicit HTTP Request nodes are required to create an end-to-end process.
2. **WF02 uses mock output:** The active path returns a fixed HVAC example; Gemini is disconnected. Connect the Gemini node, route valid prompts to it, handle its response, and test model/API availability before describing this as live AI analysis.
3. **WF02 invalid-input response is incomplete:** The false branch of the issue-description check is not connected to the webhook response node.
4. **WF03 does not persist assignments:** It returns a candidate technician but never updates the matching `Maintenance_Requests` row.
5. **WF04 does not persist statuses:** It validates an input value but never updates the corresponding ticket row.
6. **No technician notification:** WF03 does not send an email, Telegram, or other dispatch notification.
7. **No explicit webhook authentication:** The supplied webhook nodes do not configure authentication. Protect endpoints before exposing them.
8. **No formal ticket state-transition rules:** WF04 validates the allowed values but does not check whether a transition is permitted from the ticket's current state.
9. **Human review is only a flag:** Critical requests are flagged, but there is no approval queue or approval/rejection workflow.
10. **Rule-based priority can miss context:** WF01 uses keyword patterns. Criticality should be reviewed by a responsible person and not treated as a substitute for safety procedures.
11. **Limited error handling:** Add clear error branches for Google Sheets failures, model/API errors, and malformed requests.
12. **Shared data needs explicit configuration:** The JSON exports contain placeholders and the separate workflows do not automatically share execution data. Configure the same spreadsheet deliberately and pass ticket IDs and results between endpoints.

## Current Project Status

The repository contains four individually built workflow exports and screenshots. WF01 can intake and save a ticket; WF02 currently demonstrates mock analysis; WF03 can return a technician match; and WF04 can validate a status payload. The complete intake-to-analysis-to-assignment-to-status process is **not yet integrated or production-ready**.

## Author

**Jasvant Malvi**

- GitHub: https://github.com/jasvantMalvi
- LinkedIn: https://www.linkedin.com/in/jasvant-malvi-810305429/
