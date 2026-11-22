# Monthly Client Work & Billing Automation System

An n8n workflow for turning monthly client work-log entries into client-wise billing totals, PDF invoices, and a consolidated Telegram summary. It uses Google Sheets as the source of client and work records, PDF.co/PDFOps invoice generation, and Telegram for the run trigger and document delivery.

> **Execution model:** The supplied workflow is initiated through a Telegram message. It does not contain a monthly Schedule Trigger. The billing calculation selects records for the current month when the workflow runs.

## Key Features

- Stores client details and work entries in Google Sheets
- Reads client and monthly work records from separate spreadsheet tabs
- Filters work entries to the current month and year
- Groups eligible work records by client
- Calculates client-wise totals using the logged amount or quantity × rate
- Removes exact duplicate work-item objects before billing
- Prepares one invoice data object per client
- Generates PDF invoices through the PDFOps invoice node
- Sends generated PDF documents to a configured Telegram chat
- Appends invoice/report metadata to the `Monthly_Reports` sheet
- Sends a consolidated monthly billing summary through Telegram
- Responds to the exact greetings `Hi` and `Hello` with a welcome message

## Workflow Architecture

The workflow has two paths after the Telegram trigger.

### 1. Telegram entry and welcome path

1. `Telegram Trigger` receives a Telegram message.
2. `If` checks whether the message text is exactly `Hi` or `Hello`.
3. If it matches, `Send a text message` sends the configured welcome response.
4. If it does not match, the workflow continues into the billing pipeline.

**Important:** The IF condition is case-sensitive and does not check for a dedicated billing command. Any message other than exactly `Hi` or `Hello` can start the billing process. Use this workflow only with the intended admin chat until this trigger condition is tightened.

### 2. Monthly billing and invoice path

1. `Get Client Details` reads the `Clients` sheet.
2. `Get Monthly Work Records` reads the `Work_Log` sheet.
3. `Calculate Client-Wise Billing`:
   - matches work records to client details using `Client_ID` (or the fallback `Client` field);
   - parses supported date formats;
   - keeps records for the current month and year;
   - uses `Amount` when supplied, otherwise calculates `Qty × Rate`;
   - groups work items by client;
   - removes exact duplicate work-item objects;
   - calculates each client's total and work-item count.
4. `Prepare Invoice Data` builds invoice details for each client, including invoice number, invoice date, INR currency, item descriptions, quantities, unit prices, and client contact details.
5. `Generate an invoice PDF` creates a PDF invoice using the configured PDFOps node.
6. `Send a document` sends the generated PDF to the configured Telegram chat.
7. `Save Monthly Report History` appends a report record to the `Monthly_Reports` sheet, including the client, billing period, total amount, status, and Telegram file ID.
8. `Prepare Monthly Summary` calculates consolidated totals and creates a client-wise breakdown.
9. `Send Monthly Summary` sends the consolidated summary through Telegram.

## Google Sheets Structure

The workflow uses one spreadsheet with three tabs.

### `Clients`

The client lookup code uses these fields:

| Column | Purpose |
|---|---|
| `Client_ID` | Client identifier used to match work-log records |
| `Client_Name` | Name printed on the invoice and monthly summary |
| `Email` | Included in the invoice's client contact lines |
| `Phone` | Included in the invoice's client contact lines |

The code also accepts `Client` as a fallback identifier if `Client_ID` is not present.

### `Work_Log`

The billing calculation reads these fields:

| Column | Purpose |
|---|---|
| `Client_ID` | Links a work entry to a client |
| `Date` | Determines whether the entry belongs to the current billing month |
| `Project` | Project name included in the invoice line description |
| `Work_Description` or `Work` | Description of the completed work |
| `Pricing_Type` | Pricing method/label included in the line description |
| `Unit` | Work unit included in the line description |
| `Qty` | Quantity used for billing |
| `Rate` | Unit rate used when `Amount` is blank |
| `Amount` | Explicit line amount; takes precedence when populated |

The code accepts dates in either `YYYY-M-D` / `YYYY-MM-DD` format or `D-MMM` format, such as `6-Aug`. The short format is interpreted using the current year.

### `Monthly_Reports`

The report-history node appends these fields:

| Column | Purpose |
|---|---|
| `Report_ID` | Generated as `RPT-<YYYY-MM>-<Client_ID>` |
| `Client_ID` | Client associated with the invoice |
| `Month` | Billing period in `YYYY-MM` format |
| `Total_Amount` | Calculated client billing total |
| `Status` | Set to `Sent` by the workflow |
| `Telegram_File_ID` | File ID returned by Telegram after sending the PDF |

## Invoice Output

The supplied invoice-preparation code uses:

- **Sender name:** `CAD Solutions`
- **Currency:** INR
- **Invoice number:** `INV-<YYYY-MM>-<Client_ID>`
- **PDF filename:** `<Client_ID>_<YYYY-MM>_Invoice.pdf`
- **Client details:** Client name, phone, and email when available
- **Line items:** Project, work description, pricing type, unit, quantity, and unit price

The workflow currently sends generated PDF files to one configured Telegram chat. Although the client's email and phone are included in the invoice data, the supplied workflow does **not** email invoices directly to clients.

## Workflow File

[Download n8n Workflow JSON](./workflow.json) — Sanitized workflow export for importing into n8n.

## Screenshot

### n8n Workflow Overview

![Monthly Client Work and Billing Automation Workflow](./screenshots/workflow-overview.png)

[Open the screenshots folder](./screenshots/)

## Setup

1. Import `workflow.json` into your n8n instance. The exported workflow is inactive by default.
2. Create or select a Google Spreadsheet with the `Clients`, `Work_Log`, and `Monthly_Reports` tabs.
3. Replace `YOUR_GOOGLE_SHEET_ID` in the Google Sheets nodes and select the matching tabs.
4. Connect your Google Sheets credentials.
5. Connect your PDFOps/PDF.co credential to `Generate an invoice PDF`.
6. Connect your Telegram credentials to the Telegram Trigger and Telegram send nodes.
7. Replace `YOUR_TELEGRAM_CHAT_ID` in `Send a document`, `Send Monthly Summary`, and `Send a text message` with the intended administrator chat ID.
8. Ensure client IDs in `Clients` match the client IDs in `Work_Log`.
9. Enter work dates in one of the supported formats and check that quantity, rate, and amount values are numeric.
10. Test using a small set of work-log records for the current month. Confirm the generated invoice totals, PDF file, report-history row, and Telegram summary before using real billing data.

The workflow JSON in this repository uses placeholders for the Google Sheets ID and Telegram chat ID and omits account credential references. Reconnect your own credentials after importing.

## Current Limitations and Recommended Improvements

The following limitations are visible in the supplied workflow:

- **No monthly scheduler:** The main billing path starts from the Telegram Trigger, not a monthly Schedule Trigger. Add a Schedule Trigger if the process should run automatically at month end.
- **Unsafe Telegram trigger condition:** Only exact, case-sensitive `Hi` and `Hello` messages take the welcome branch. Other messages—including typos or unrelated text—start billing. Add an explicit command such as `/generate_monthly_bill` and an authorized-chat check.
- **Billing period is always the runtime's current month:** The code uses `new Date()` to determine the year and month. It does not accept a user-selected historical billing period.
- **Date formats are limited:** Dates such as `DD/MM/YYYY` are not handled by the supplied parser. Invalid calendar dates are not explicitly validated before JavaScript date construction.
- **Exact duplicate rows are discarded:** The code removes rows with identical date, project, description, pricing type, unit, quantity, rate, and amount. If two identical entries represent separate legitimate work, one may be excluded from billing.
- **Reruns can create repeated invoices and history rows:** The PDF filename and invoice number are based on client and billing month, but the workflow does not check whether that client/month has already been invoiced. The report-history node uses append, so rerunning the workflow can append duplicate history records and send another PDF.
- **Invoices are sent to a fixed Telegram destination:** The workflow sends every PDF and the monthly summary to one configured chat. It does not deliver invoices to each client's email address.
- **Invoice fields are basic:** The supplied invoice data includes sender name, client contact details, INR currency, line items, invoice number, date, and a note. It does not calculate GST/tax, discounts, payment terms, or statutory invoice fields.
- **Runtime timezone affects month selection:** The current month and year are taken from the n8n runtime's local date. Configure and test the n8n timezone so month-boundary billing runs use the intended business timezone.
- **No empty-month completion path is guaranteed:** If the monthly calculation returns no client items, downstream invoice nodes may receive no items and the summary path may not execute as intended. Add an explicit no-records branch if a zero-activity report is required.

Review invoice totals, duplicate handling, billing dates, and tax requirements before using generated PDFs as customer-facing accounting documents.

## Repository Structure

```text
Monthly-Client-Work-and-Billing-Automation-System/
├── screenshots/
│   ├── README.md
│   └── workflow-overview.png
├── README.md
└── workflow.json
```
