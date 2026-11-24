# LinkedIn Job Alert Agent

An automated daily job-digest workflow built with n8n, Apify, OpenAI GPT-5 Mini, Google Sheets, and Gmail. It runs on a daily schedule, retrieves up to 10 records from a configured Apify dataset, standardizes the job fields, generates structured two-sentence job summaries, stores the analyzed jobs in Google Sheets, and sends a consolidated HTML email.

## Workflow Overview

The supplied export contains **16 nodes in total**: 9 functional nodes and 7 sticky-note annotations.

```text
Daily Job Alert Trigger (9:00 AM)
              ↓
Fetch LinkedIn Job Data (Apify dataset, max 10)
              ↓
Normalize Job Data (JavaScript)
              ↓
AI Job Analyzer (GPT-5 Mini + Structured Output Parser)
              ↓
       ┌──────┴───────────────┐
       ↓                      ↓
Save Jobs to Google       Build Daily Email HTML
Sheets (append)                 ↓
                           Send Daily Job Digest
                                (Gmail)
```

## Main Workflow Stages

### 1. Daily Schedule

`Daily Job Alert Trigger` starts the workflow every day at **9:00 AM**, using the schedule configured in the exported workflow. The schedule follows the timezone configured in the n8n instance.

### 2. Apify Dataset Retrieval

`Fetch LinkedIn Job Data` uses the Apify integration to retrieve records from a configured dataset. The node requests a maximum of **10 items**.

Important distinction: the workflow reads from an existing Apify dataset ID. It does not start an Apify actor or scrape LinkedIn itself in this n8n workflow. The dataset must be populated and refreshed by an Apify actor or another upstream process. If the dataset is not refreshed, the daily workflow may process the same records again.

### 3. Normalize Job Data

`Normalize Job Data` uses JavaScript to standardize each fetched record into these fields:

- `title`
- `company`
- `location`
- `link`
- `description`
- `posted_date`

The code uses fallback source keys for company, description, and posting date where available.

### 4. AI Job Analysis

`AI Job Analyzer` receives each normalized job and asks GPT-5 Mini to produce an accurate professional summary in two sentences. The prompt instructs the model to preserve the original application URL and not invent missing information.

`Job JSON Parser` enforces a structured output example with these fields:

- `job_title`
- `company`
- `location`
- `apply_link`
- `summary`
- `posted_date`
- `notification`

The workflow uses the structured output fields downstream. The `notification` field is defined in the parser schema example, but the current storage and email branches construct their own output formats and do not use that field.

### 5. Google Sheets Storage

`Save Jobs to Google Sheets` appends each analyzed job to the `Sheet1` tab of the configured spreadsheet.

The mapped columns are:

| Google Sheets column | AI output field |
|---|---|
| Job Title | `output.job_title` |
| Company | `output.company` |
| Location | `output.location` |
| Apply Link | `output.apply_link` |
| Summary | `output.summary` |
| Date | `output.posted_date` |

The node uses the **append** operation and has no matching columns configured. Every run can append another copy of a previously processed job.

### 6. Consolidated Email Digest

`Build Daily Email HTML` loops through the analyzed job items and creates one HTML card per item, including title, company, location, summary, and an Apply Now link.

`Send Daily Job Digest` sends the generated HTML as one email with a date-based subject line.

The Gmail node uses a fixed recipient in the original workflow. The public export replaces it with a setup placeholder.

## Example Output

The supplied screenshots show:
- An Apify dataset containing company and job-related fields.
- A Google Sheets table with job title, company, location, application link, summary, and posting date.
- A consolidated Gmail digest containing multiple job cards and Apply Now links.

The sample email includes roles located in Ahmedabad, Gujarat, India. This is the location shown in the sample records; the workflow itself does not apply an Ahmedabad-only location filter in the supplied JSON.

## Technology Stack

- **n8n** — scheduling and workflow orchestration
- **Apify** — retrieval of records from an existing dataset
- **OpenAI GPT-5 Mini** — job-description summarization
- **Structured Output Parser** — consistent AI response structure
- **JavaScript** — normalization and HTML email generation
- **Google Sheets** — job record storage
- **Gmail** — consolidated daily job digest

## Project Files

- [Workflow JSON](./workflow.json) — sanitized export of the actual n8n workflow.
- [Screenshots](./screenshots/README.md) — workflow, Apify dataset, spreadsheet, and email examples.

## Setup

1. Import `workflow.json` into your n8n instance. The public export is inactive.
2. Connect your Apify account credentials to `Fetch LinkedIn Job Data`.
3. Replace `REPLACE_WITH_YOUR_APIFY_DATASET_ID` with the dataset ID that contains your job records.
4. Ensure that an upstream Apify actor or process refreshes that dataset before the daily n8n run. This workflow only reads the dataset.
5. Connect your OpenAI credentials to `GPT Job Analysis Model`.
6. Create a Google Spreadsheet with a `Sheet1` tab and these exact headers: `Job Title`, `Company`, `Location`, `Apply Link`, `Summary`, and `Date`.
7. Replace `REPLACE_WITH_YOUR_GOOGLE_SHEET_ID` with the target spreadsheet ID and connect Google Sheets credentials.
8. Connect Gmail credentials to `Send Daily Job Digest`.
9. Replace `REPLACE_WITH_YOUR_EMAIL` with the inbox that should receive the digest.
10. Execute the workflow manually with a test dataset and verify the number and content of jobs in both Google Sheets and Gmail.
11. Activate the workflow only after confirming the schedule timezone and upstream dataset refresh schedule.

## Important Limitations and Checks

### 1. The workflow does not scrape LinkedIn itself

The Apify node reads from a fixed dataset ID. No Apify actor-start or actor-run node is present in the supplied workflow. Job retrieval therefore depends on an external process populating that dataset.

### 2. The same jobs can be stored repeatedly

The Google Sheets operation is `append`, and `matchingColumns` is empty. The workflow has no duplicate detection or update logic. If the dataset contains the same records on consecutive days, duplicate rows can accumulate.

### 3. The dataset is capped at 10 items

The Apify node requests a maximum of 10 dataset items. It does not implement pagination or a configurable larger result set in the supplied export.

### 4. No location or role filtering is implemented in n8n

The JavaScript normalization step maps fields, but the supplied workflow does not filter jobs by city, role, experience level, keywords, or remote status. Any such narrowing must happen upstream in the Apify dataset or be added as a workflow step.

### 5. The sample email subject does not appear to match the intended date formatting

The supplied screenshot shows a subject similar to `Daily Job Alert - Nov 4, 2026-11-YYYY`, while the node expression uses `{{$now.format('DD-MM-YYYY')}}`. Verify the rendered subject with a test execution and correct the format if needed.

### 6. The workflow sends a digest regardless of whether jobs are relevant

There is no scoring, candidate-profile matching, relevance threshold, or empty-result branch in the supplied graph. The AI writes summaries; it does not evaluate whether a role fits a particular candidate.

### 7. No per-job delivery status or application tracking

The workflow records job data, but it does not track whether the user viewed, saved, applied to, or rejected a job.

### 8. AI summaries depend on source data quality

The prompt tells the model not to invent missing information, but there is no separate fact-checking step. Check the original job posting before applying, especially for salary, experience, eligibility, and location details not present in the dataset.

### 9. Email HTML is constructed from model-generated values

The Code node inserts AI output into HTML. The supplied code does not escape HTML special characters in those values. For production use, escape untrusted text before inserting it into HTML and validate application URLs before using them in links.

### 10. No dedicated error-handling path

The workflow does not show separate retry or alert branches for Apify dataset failures, OpenAI errors, Google Sheets errors, or Gmail delivery failures.

## Current Project Status

The supplied workflow automates scheduled retrieval from an Apify dataset, AI-assisted job summarization, Google Sheets logging, and consolidated Gmail delivery. It is a daily job-digest prototype, not a complete job-search platform: source refresh is external, duplicates are not prevented, job relevance is not scored, and application progress is not tracked.

## Author

<p align="left">
  <b>Jasvant Malvi</b><br>
  <a href="https://github.com/jasvantMalvi">GitHub</a> ·
  <a href="https://www.linkedin.com/in/jasvant-malvi-810305429/">LinkedIn</a>
</p>
