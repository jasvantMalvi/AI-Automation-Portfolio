# AI Invoice Analyzer Bot

An AI-powered invoice extraction and validation workflow built using n8n, Gmail, OpenAI, JavaScript, and Google Sheets.

The workflow receives invoice attachments through Gmail, extracts key invoice information using AI, analyzes the extracted data for missing fields and inconsistencies, generates an HTML report, records validation results, and emails the report to the original sender.

## Project Overview

Manual invoice processing can involve repetitive data extraction, validation, record keeping, and communication.

This project demonstrates how AI and workflow automation can simplify those steps through an email-triggered invoice analysis process.

## Key Features

- Automatic Gmail invoice email monitoring
- Invoice attachment retrieval
- AI-powered invoice image analysis
- Structured JSON data extraction
- Vendor and invoice number checks
- Invoice date and due date consistency checks
- Subtotal, tax, and total amount validation
- Potential fraud indicator assessment
- Invoice risk-level assessment
- Invoice quality scoring logic
- AI-generated recommendations
- HTML invoice validation report
- Google Sheets validation record storage
- Automated email report delivery

## Invoice Data Extracted

The image analysis prompt requests these fields:

- Vendor name
- Invoice number
- Invoice date
- Due date
- Customer name
- Subtotal
- Discount
- Tax
- Total amount
- Payment method
- Currency

Missing information should be returned as null rather than guessed.

## Workflow Architecture

1. **Gmail Trigger:** Checks for incoming emails every minute.
2. **Get a Message:** Retrieves the email and downloads its invoice attachment.
3. **Analyze Image:** Uses GPT-4o-mini to extract invoice information into JSON.
4. **AI Agent:** Reviews the extracted data for completeness, logical consistency, risk indicators, and recommendations.
5. **Code in JavaScript:** Prepares an HTML invoice validation report.
6. **Append Row in Sheet:** Stores validation status, risk level, issues, recommendations, and processing timestamp.
7. **Send a Message:** Emails the generated report to the original sender.

The OpenAI Chat Model is connected to the AI Agent for analysis.

## Technology Stack

| Technology | Purpose |
|---|---|
| n8n | Workflow automation |
| Gmail | Invoice email intake and report delivery |
| OpenAI GPT-4o-mini | Invoice image analysis and validation |
| JavaScript | Report generation and data processing |
| Google Sheets | Validation record storage |

## Project Files

- `workflow.json` — Sanitized n8n workflow export
- `screenshots/` — Workflow overview and execution evidence

## Known Limitations

- The current JavaScript risk-level comparisons use title-case values, while the AI prompt requests uppercase values. This can cause the calculated score to fall through to its default value.
- The HTML report contains a fixed successful-validation message, which should be made conditional on the actual validation status.
- The displayed score is derived from risk level; it is not an independently measured confidence score.
- Google Sheets currently stores validation results and recommendations, not every extracted invoice field.
- The workflow requires testing with different invoice formats and missing or inconsistent values.

These issues should be corrected and retested before describing the workflow as production-ready.

## Setup Requirements

- n8n instance
- Gmail OAuth2 credentials
- OpenAI API credentials
- Google Sheets credentials
- A Google Sheet with the required validation columns

Import the workflow and configure your own credentials, spreadsheet ID, and sheet settings before testing.

## Privacy Note

Use synthetic invoice data for public demonstrations. Never publish real customer names, email addresses, bank details, tax identifiers, or invoice attachments.

## Author

**Jasvant Malvi**

- GitHub: https://github.com/jasvantMalvi
- LinkedIn: https://www.linkedin.com/in/jasvant-malviya-810305429
