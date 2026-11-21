# Project Screenshots

This folder contains visual documentation of the **AI Powered Personal Assistant in Telegram – Audio and Text**, built using n8n, Telegram, OpenAI, Gmail, Notion, Google Calendar, and Google Contacts.

## Workflow Overview

**File:** `workflow-overview.png`

The screenshot shows the complete automation workflow, including:

- Receiving text and voice messages through Telegram
- Normalizing incoming Telegram messages using JavaScript
- Routing text and voice messages through an IF node
- Downloading Telegram voice messages
- Transcribing audio into text
- Processing user requests through an AI Personal Assistant
- Maintaining conversation context using Simple Memory
- Creating contacts in Google Contacts
- Retrieving and sending Gmail messages
- Creating and retrieving Notion tasks
- Creating and retrieving Google Calendar events
- Sending AI-generated responses back through Telegram

## Notes

- Both text and voice inputs are supported.
- Voice messages are transcribed before being processed by the AI Agent.
- The assistant uses connected tools to perform supported actions.
- Email sending and calendar event creation require user confirmation according to the agent instructions.
- Tool availability depends on the connected accounts and configured permissions.
- Configure your own credentials and account references before execution.
- Never publish API keys, Telegram bot tokens, private email addresses, or personal data.

