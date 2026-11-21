# AI Document Summarizer Agent

An AI-powered document processing and summarization workflow built using n8n.

The system automates document intake, text extraction, AI-powered summarization, and delivery of the generated summary.

## Key Features

- Automated document intake from Google Drive
- Document download and text extraction
- AI-powered document summarization
- Structured summary generation
- Automated summary delivery
- Error handling for document processing

## Workflow Overview

1. Detect a new document in Google Drive.
2. Download the document.
3. Extract its text content.
4. Process the extracted content using an AI model.
5. Generate a concise summary.
6. Deliver the generated summary through the configured integration.

## Technology Stack

- n8n — Workflow automation
- Google Drive — Document storage and intake
- OpenAI API — AI summarization
- JavaScript — Data processing

## Project Structure

```text
AI-Document-Summarizer-Agent/
├── README.md
├── workflow.json
└── screenshots/
    ├── README.md
    └── workflow-overview.png
```

## Setup Requirements

- n8n instance
- Google Drive credentials
- AI model API credentials
- Configured document extraction and delivery nodes

## Important Notes

- Supported document formats depend on the configured extraction nodes.
- Summary quality depends on the extracted text.
- Configure your own credentials before importing and testing.
- Never publish API keys, access tokens, or private documents.

## Author

**Jasvant Malvi**

- GitHub: https://github.com/jasvantMalvi
- LinkedIn: https://www.linkedin.com/in/jasvant-malvi-810305429/

