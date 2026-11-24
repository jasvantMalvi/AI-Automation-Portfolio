# Auto Presentation Generator Agent

An n8n workflow that turns a presentation brief into a six-slide Google Slides deck. It uses GPT-5 Mini to generate structured slide content, copies a master template, replaces slide placeholders, logs the presentation in Google Sheets, and emails the link to the requester.

## Project snapshot

| Item | Details |
|---|---|
| Workflow platform | n8n |
| AI model | OpenAI GPT-5 Mini |
| Presentation output | Google Slides |
| Tracking | Google Sheets |
| Delivery | Gmail (HTML email) |
| Input | n8n form |
| Example run | IR Spectroscopy — Education — 6 slides — Students — English |

## Workflow overview

![Auto Presentation Generator workflow architecture](./screenshots/workflow-architecture.svg)

```text
Presentation Request Form
          ↓
Collect Presentation Request (AI Agent)
   ├── OpenAI Content Generation Model (GPT-5 Mini)
   └── Validate Presentation JSON (Structured Output Parser)
          ↓
Duplicate Master Presentation (Google Drive)
          ↓
Populate Google Slides
          ↓
Share Presentation Access
          ↓
Log Presentation in Google Sheets
          ↓
Email Generated Presentation (Gmail)
```

The exported workflow contains **15 nodes**: 9 functional nodes and 6 sticky-note annotations.

## Input form

The form collects six fields: Presentation Topic, Presentation Type (Education / Official / Professional), Number of Slides, Target Audience (Students / Boss / Family), Presentation Language (English / Hindi / Gujarati), and Email Address.

## AI-generated slide structure

The Structured Output Parser expects a presentation title and subtitle, plus titles and point strings for slides 2–6. The prompt defines six fixed slides: Cover, Introduction, Key Concepts, Detailed Explanation, Applications, and Conclusion. Slides 2–6 are instructed to contain exactly three concise bullet points each.

## Workflow stages

1. **Collect request:** capture topic, type, slide count, audience, language and recipient email.
2. **Generate content:** GPT-5 Mini creates structured slide content; the Structured Output Parser validates the expected JSON structure.
3. **Copy template:** Google Drive creates a separate copy of the master presentation in the configured destination folder.
4. **Populate slides:** Google Slides replaces 11 title/content placeholders for the cover title and slides 2–6.
5. **Share:** grants viewer access to the generated presentation.
6. **Log:** appends creation date, topic, type, requested slide count and Google Slides URL to Google Sheets.
7. **Deliver:** Gmail sends an HTML email with presentation details and a link.

The public workflow export corrects one issue in the original: the share step now targets the ID returned by the template-copy node rather than a hard-coded test-presentation ID.

## Example run

The supplied screenshots document an **IR Spectroscopy** presentation in English for Students, with Education type and six slides. The Google Sheets screenshot shows a presentation log entry, and the Gmail screenshot shows the “Your AI-Generated Presentation is Ready!” email. The downloadable project package includes the supplied screenshots in higher resolution.

## Limitations found in the supplied workflow

These findings are based on the uploaded n8n JSON and screenshots:

1. **Requested slide count is ignored by the prompt.** The form collects Number of Slides, but the prompt explicitly requests six fixed slides and does not reference the submitted number. The log stores the requested number, which could differ from the actual output.
2. **The generated subtitle is not inserted.** The parser returns `presentation_subtitle`, but the Google Slides replacement list does not map it to a template placeholder. The generated-cover screenshot still shows “Click to add subtitle”. Add a subtitle placeholder to the master slide and map it.
3. **Presentation type is not fully enforced.** The selected type is passed to the AI, but both the prompt and system message still define the presentation as educational. “Official” and “Professional” may not reliably change the format.
4. **Original share node had a fixed test file ID.** The public `workflow.json` corrects this by using the copied presentation ID. Use the public export or make the same change in the original n8n workflow.
5. **Anyone-with-the-link sharing is broad.** The source workflow grants `anyone` the `reader` role. Change this for private or organization-only presentations.
6. **No explicit error-handling route is shown.** There are no dedicated failure notifications or recovery branches for AI generation, copying, slide replacement, sharing, logging or email delivery.
7. **The exported workflow is inactive and contains resource placeholders.** Reconnect credentials and configure your own Google resource IDs before running it.

## Setup

1. Import `workflow.json` into n8n. The export is inactive by design.
2. Reconnect OpenAI, Google Drive, Google Slides, Google Sheets and Gmail credentials.
3. In **Duplicate Master Presentation**, set `REPLACE_WITH_MASTER_GOOGLE_SLIDES_TEMPLATE_ID` and `REPLACE_WITH_GOOGLE_DRIVE_DESTINATION_FOLDER_ID`.
4. In **Log Presentation in Google Sheets**, set `REPLACE_WITH_GOOGLE_SHEETS_SPREADSHEET_ID`. Confirm the sheet has columns: Created Date, Presentation Topic, Presentation Type, Number of Slides, Presentation URL.
5. Keep the dynamic file ID expression in **Share Presentation Access** so the newly copied deck is shared.
6. Decide whether anyone-with-the-link viewer access is suitable.
7. Add a `{{PRESENTATION_SUBTITLE}}` placeholder to the master slide and a replacement entry if you want the generated subtitle inserted.
8. Either update the prompt to use the requested slide count or remove/rename that form field.
9. Test a non-confidential topic and verify the slide count, subtitle, permissions, Sheets row and emailed link before activating the workflow.

## Repository contents

```text
Auto-Presentation-Generator-Agent/
├── README.md
├── workflow.json
└── screenshots/
    ├── README.md
    └── workflow-architecture.svg
```

The downloadable project ZIP supplied with this portfolio update also contains six curated screenshots: workflow overview, master template, generated cover, generated conclusion, Google Sheets log and Gmail delivery. Email identity text and the live Slides URL are obscured in those public copies.

## Author

**Jasvant Malvi**

- GitHub: https://github.com/jasvantMalvi
- LinkedIn: https://www.linkedin.com/in/jasvant-malvi-810305429/
