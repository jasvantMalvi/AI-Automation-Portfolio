# AI KYC Customer Verification

A document intake and verification workflow demonstration built using n8n, Telegram, Google Sheets, and JavaScript.

The workflow receives identity documents through Telegram, checks file types, creates KYC case records, processes identity-related fields, evaluates risk conditions, and routes cases toward approval or manual review.

## Project Overview

The workflow demonstrates how parts of a KYC (Know Your Customer) intake process can be organized and automated.

It includes document submission, file validation, case tracking, identity-field checks, duplicate lookup, risk scoring, decision routing, and audit logging.

## Key Features

- Telegram-based document submission
- PDF, JPG, and PNG file-type validation
- Telegram file retrieval
- Automatic KYC case ID generation
- Google Sheets case tracking
- OCR-related processing structure
- Identity name and document-number checks
- Duplicate document lookup
- Address verification structure
- Risk score calculation
- Approval or manual-review routing
- Audit log generation
- Telegram status notifications

## Workflow Architecture

1. **Document Intake:** Telegram Trigger receives a document or message.
2. **Message Normalization:** Extracts chat, file, and message information.
3. **File Validation:** Checks file availability and accepts PDF, JPG, and PNG formats.
4. **KYC Case Creation:** Generates a unique KYC request ID and records the case.
5. **Document Processing:** Prepares identity fields and OCR-related data.
6. **Identity Checks:** Evaluates name and document-number conditions.
7. **Duplicate Check:** Looks up document records in Google Sheets.
8. **Address Check:** Includes an address-verification stage.
9. **Risk Assessment:** Calculates a risk score and assigns a risk level.
10. **Decision Routing:** Routes cases to approval or manual review.
11. **Audit Logging:** Generates and stores an audit record.
12. **User Notification:** Sends the final status through Telegram.

## Technology Stack

| Technology | Purpose |
|---|---|
| n8n | Workflow automation |
| Telegram Bot API | Document intake and notifications |
| Google Sheets | KYC records and audit logs |
| JavaScript | Data normalization and risk calculations |
| HTTP Request | External-service integration structure |

## Important Implementation Limitations

This repository contains a workflow demonstration, not a production-ready identity verification system.

The current export includes simulated or incomplete verification stages:

- The OCR preparation node uses hardcoded test identity values.
- The OCR HTTP Request node is not connected into the active execution path.
- The name comparison currently compares values derived from the same extracted name rather than independently verified customer data.
- The duplicate-check branch currently sets the duplicate flag to true.
- Address verification currently uses a preset test value; its HTTP verification nodes are not connected into the active path.

These stages must be corrected and tested before treating the workflow as a real KYC verification solution.

## Setup Requirements

- n8n instance
- Telegram Bot credentials
- Google Sheets credentials
- KYC case and audit spreadsheets
- Configured OCR and address-verification services, if implemented

Import the workflow and configure your own credentials and spreadsheet references. Use synthetic documents and test records only.

## Privacy and Security

Never use real identity documents, Aadhaar numbers, PAN numbers, passport details, or customer personal information in public demonstrations.

## Project Files

- `workflow.json` — Sanitized n8n workflow export
- `screenshots/` — Workflow architecture and test execution screenshots

## Author

**Jasvant Malvi**

- GitHub: https://github.com/jasvantMalvi
- LinkedIn: https://www.linkedin.com/in/jasvant-malvi-810305429

---

## Workflow Demonstration

![KYC Workflow and Successful Execution](screenshots/Successful%20Execution.jpeg)

## Workflow Export

[View sanitized n8n workflow](workflow.json)
