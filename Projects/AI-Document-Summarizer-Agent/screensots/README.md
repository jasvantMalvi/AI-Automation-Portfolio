# Project Screenshots

This folder contains visual documentation of the **AI Document Summarizer Agent**, built using n8n, Google Drive, Google Docs, OpenAI, and Slack.

## Workflow Overview

**File:** `workflow-overview.png`

The screenshot shows the complete automation workflow, including:

- Monitoring a Google Drive folder for new documents
- Retrieving document content from Google Docs
- Analyzing document content using an AI agent
- Processing the document with the OpenAI GPT-5 mini model
- Structuring the summary using a predefined JSON output parser
- Sharing the generated summary and insights through Slack

## Notes

- The workflow is configured to monitor a specific Google Drive folder.
- Document analysis uses the content retrieved from Google Docs.
- AI-generated summaries follow a structured output format.
- Slack is used to deliver the generated analysis report.
- The screenshot shows the workflow execution indicators.
- Configure your own credentials and folder references before execution.
- Do not publish private documents, API keys, or access tokens.

