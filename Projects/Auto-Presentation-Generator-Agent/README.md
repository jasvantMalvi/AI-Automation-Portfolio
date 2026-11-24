# Auto Presentation Generator Agent

An n8n automation that turns a presentation request form into a six-slide Google Slides presentation. It uses an AI agent to generate structured slide content, duplicates a master Slides template, replaces slide placeholders, shares the generated deck, records its details in Google Sheets, and emails the requester a presentation link.

> **Project status:** The supplied workflow was inactive in the uploaded export. The screenshots show a successful example for an *IR Spectroscopy* presentation, including a generated Google Slides deck, a Google Sheets log entry and a delivery email. The exported workflow is provided as a sanitized, inactive template that must be reconnected to your own Google and OpenAI accounts before use.

## Workflow overview

```text
Presentation Request Form
          |
          v
Collect Presentation Request (AI Agent)
      /                 \\
OpenAI Model        Structured Output Parser
          |
          v
Duplicate Master Presentation
          |
          v
Populate Google Slides
          |
          v
Share Presentation Access
          |
          v
Log Presentation in Google Sheets
          |
          v
Email Generated Presentation
```

The supplied workflow contains **15 nodes**: 9 functional nodes and 6 sticky-note annotations.

## How it works

### 1. Collect presentation requirements

The n8n form collects:

| Input | Available values / format |
|---|---|
| Presentation Topic | Text |
| Presentation Type | Education, Official, Professional |
| Number of Slides | Number |
| Target Audience | Students, Boss, Family |
| Presentation Language | English, Hindi, Gujarati |
| Email Address | Email |

### 2. Generate structured content

The `Collect Presentation Request` AI Agent uses the connected OpenAI model, configured in the export as **GPT-5 Mini**, and the `Validate Presentation JSON` structured output parser.

The prompt defines six fixed slides:

1. Cover — presentation title and subtitle
2. Introduction — title and three bullet points
3. Key Concepts — title and three bullet points
4. Detailed Explanation — title and three bullet points
5. Applications — title and three bullet points
6. Conclusion — title and three bullet points

Content is requested in the selected language and is intended to be concise and suitable for the selected audience.

**Important:** Although the form collects a `Number of Slides` value, the supplied AI prompt and output schema generate exactly six slides. The entered slide count does not currently control the number of slides.

### 3. Duplicate the master presentation

`Duplicate Master Presentation` copies the configured Google Slides master template into the selected Drive folder. A separate copy is created for each request.

### 4. Populate the slides

`Populate Google Slides` replaces the predefined placeholders with the generated presentation title and titles and bullet points for slides 2–6.

The master template needs to contain the matching placeholders:

- `{{PRESENTATION_TITLE}}`
- `{{SLIDE2_TITLE}}` and `{{SLIDE2_POINTS}}`
- `{{SLIDE3_TITLE}}` and `{{SLIDE3_POINTS}}`
- `{{SLIDE4_TITLE}}` and `{{SLIDE4_POINTS}}`
- `{{SLIDE5_TITLE}}` and `{{SLIDE5_POINTS}}`
- `{{SLIDE6_TITLE}}` and `{{SLIDE6_POINTS}}`

### 5. Share the generated presentation

`Share Presentation Access` grants **viewer access to anyone with the link**. This is convenient for delivery, but it means the generated presentation is not restricted to only the recipient. Change the permission settings if the content is private.

**Fix included in this public export:** The uploaded workflow used a fixed presentation ID in the sharing node. The sanitized version now references the presentation created by `Duplicate Master Presentation`, so it shares the current request's generated deck.

### 6. Log the request

`Log Presentation in Google Sheets` appends these fields: Created Date, Presentation Topic, Presentation Type, Number of Slides, and Presentation URL.

### 7. Email the requester

`Email Generated Presentation` sends an HTML email to the submitted email address. It includes the topic, presentation type, slide count, audience, language and a link to open the generated Google Slides presentation.

## Technology stack

- **n8n** — workflow orchestration and form trigger
- **OpenAI GPT-5 Mini** — presentation content generation
- **Structured Output Parser** — validates the expected JSON structure
- **Google Drive** — copies the master template and manages sharing
- **Google Slides** — replaces slide placeholders with generated content
- **Google Sheets** — stores a presentation creation log
- **Gmail** — sends the generated presentation link

## Repository contents

```text
Auto-Presentation-Generator-Agent/
├── README.md
├── workflow.json
└── screenshots/
    ├── README.md
    ├── workflow-overview.png
    ├── master-template.png
    ├── template-slide-placeholders.png
    ├── generated-presentation-cover.png
    ├── generated-presentation-conclusion.png
    ├── google-sheets-log.png
    └── delivery-email.png
```

## Setup

1. Import `workflow.json` into n8n. The exported workflow is inactive.
2. Connect your own OpenAI credential to `OpenAI Content Generation Model`.
3. Connect your Google Drive credential to `Duplicate Master Presentation` and `Share Presentation Access`.
4. Replace `REPLACE_WITH_MASTER_GOOGLE_SLIDES_TEMPLATE_FILE_ID` with your master Slides template file ID.
5. Replace `REPLACE_WITH_OUTPUT_GOOGLE_DRIVE_FOLDER_ID` with the Drive folder ID where generated decks should be stored.
6. Connect your Google Slides credential to `Populate Google Slides`.
7. Confirm the master template contains every placeholder listed above. Keep placeholder spelling and capitalization exact.
8. Connect your Google Sheets credential and replace `REPLACE_WITH_GOOGLE_SHEETS_SPREADSHEET_ID` with your presentation log spreadsheet ID. Confirm the sheet has the five expected column headers.
9. Connect your Gmail credential to `Email Generated Presentation`.
10. Review the sharing permission. The current workflow grants viewer access to anyone with the link.
11. Submit a test request and verify the generated slide content, file sharing, spreadsheet row and email before activating the workflow.

## Known limitations and checks

- **Fixed six-slide structure:** The AI prompt and parser schema are hard-coded for six slides. The form's slide-count value is logged and shown in the email, but it does not change the generated slide count.
- **Master-template dependency:** The workflow depends on a Google Slides template with the exact text placeholders. A missing or differently spelled placeholder will not be replaced as expected.
- **Public link sharing:** The current permission is `anyone` with the `reader` role. Anyone with the resulting URL can view the presentation.
- **No explicit error branch:** The supplied workflow has no dedicated error-handling route or failure notification for AI generation, template copying, placeholder replacement, sharing, logging or email delivery.
- **No content fact-checking step:** The AI output is schema-validated, but the workflow does not independently verify factual claims in generated slide content.
- **No dynamic slide-count support:** Supporting a variable number of slides requires changes to the AI prompt, structured schema and master template replacement logic.

## Example output

The supplied screenshots document an example request for **IR Spectroscopy**: Education presentation type, Students audience, English language and six requested slides. The example deck contains six slides; the workflow and screenshots also show a Google Sheets log and Gmail delivery email.

## Author

<p align="left">
  <b>Jasvant Malvi</b><br>
  <a href="https://github.com/jasvantMalvi">GitHub</a> ·
  <a href="https://www.linkedin.com/in/jasvant-malvi-810305429/">LinkedIn</a>
</p>
