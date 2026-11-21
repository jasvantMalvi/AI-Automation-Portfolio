# AI Maintenance Dispatch System

A multi-workflow maintenance request management and dispatch automation project built using n8n and Google Sheets.

The system is designed to organize maintenance requests, analyze reported issues, match technicians, and validate maintenance status through separate connected workflows.

## Key Features

- Maintenance request intake
- Structured maintenance ticket creation
- AI-style issue analysis using a mock AI response in the current version
- Technician matching
- Maintenance status validation
- Shared Google Sheets data source
- Human review flag using `requires_human_review`

## Workflow Architecture

### WF01 — Maintenance Request Intake

Receives maintenance requests and records ticket information in the shared Google Spreadsheet.

### WF02 — AI Issue Analyzer

Processes maintenance issue information and generates structured analysis. The current implementation uses mock AI output rather than a live AI model.

### WF03 — Technician Matching

Matches maintenance requests with suitable technicians based on the available workflow data.

### WF04 — Status Validation

Validates maintenance status information and supports the status-checking process.

## Technology Stack

- n8n — Workflow automation
- Google Sheets — Shared maintenance data storage
- JavaScript — Data processing and workflow logic
- AI analysis logic — Mock AI response in the current version

## Human Review

The maintenance data includes a `requires_human_review` field.

This field identifies requests that require additional human approval or safety review.

## Project Structure

```text
AI-Maintenance-Dispatch/
├── README.md
├── workflows/
│   ├── WF01-Maintenance-Request-Intake.json
│   ├── WF02-AI-Issue-Analyzer.json
│   ├── WF03-Technician-Matching.json
│   └── WF04-Status-Validation.json
└── screenshots/
    ├── README.md
    └── workflow-overview.png
```

## Current Project Status

- Four individual workflows have been built and tested.
- All four workflows use the same Google Spreadsheet.
- Cross-workflow synchronization and complete end-to-end automation remain areas for further development.
- Status persistence and additional approval, notification, and reporting features require further implementation.

## Setup Requirements

- n8n instance
- Google Sheets account and spreadsheet
- Configured Google Sheets credentials
- Imported workflow files
- Correct spreadsheet column structure

## Important Notes

- The current AI Issue Analyzer uses mock AI output.
- The project should not be considered a fully production-ready dispatch system.
- Configure your own credentials and spreadsheet references before execution.
- Never publish private spreadsheet IDs, credentials, or sensitive maintenance records.

## Author

**Jasvant Malvi**

- GitHub: https://github.com/jasvantMalvi
- LinkedIn: https://www.linkedin.com/in/jasvant-malvi-810305429/

