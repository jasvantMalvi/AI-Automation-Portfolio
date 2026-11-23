# n8n Workflow Files

This folder contains four separate webhook-triggered workflow exports for the AI Maintenance Dispatch prototype.

| File | Workflow | Webhook path | Main responsibility |
|---|---|---|---|
| `WF01-Maintenance-Request-Intake.json` | Request Intake | `POST /maintenance-request` | Validate request fields, classify basic category/priority, generate ticket ID, append to `Maintenance_Requests` |
| `WF02-AI-Issue-Analyzer.json` | Issue Analyzer | `POST /maintenance-analyze` | Prepare issue-analysis prompt and parse the response; current active route returns a fixed mock result |
| `WF03-Technician-Matching.json` | Technician Matching | `POST /maintenance-dispatch` | Read technicians and return a suitable candidate based on skills, availability, and workload |
| `WF04-Status-Validation.json` | Status Validation | `POST /maintenance-status` | Validate the requested status against an allowed list and return a JSON response |

## Important

These exports are **not automatically connected to one another**. They expose separate webhook paths; a caller or an additional orchestration workflow must invoke each stage and pass the required ticket data.

Before use:

1. Import each JSON file into n8n separately.
2. Configure your Google Sheets credentials and replace spreadsheet/worksheet placeholders.
3. Protect the webhook endpoints with authentication.
4. Test each endpoint independently.
5. Review the root project README for known limitations, including WF02 mock output and missing assignment/status persistence.

The JSON files are workflow exports, not a ready-to-run hosted service.