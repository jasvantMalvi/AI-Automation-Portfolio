# Instagram Post Automation

An n8n workflow prototype that schedules Instagram content creation, generates a structured post brief with GPT-5 Mini, creates an image using Pollinations, uploads the image to Cloudinary, creates an Instagram media container, and submits the container for publishing through the Meta Graph API.

**The supplied workflow export is inactive.** The JSON and screenshot document the workflow structure, but do not establish that the current Meta credentials, account permissions, media-processing wait, or Instagram publishing step have been successfully tested.

## Workflow Overview

The supplied export contains **15 nodes**:
- 9 functional nodes
- 6 sticky-note annotations

```text
Daily Content Scheduler (9:00 AM)
                ↓
       AI Content Generator
       (GPT-5 Mini / JSON)
                ↓
      Pollinations Image Generator
                ↓
       Upload Image to Cloudinary
                ↓
      Prepare Instagram Post Data
                ↓
       Create Instagram Media
                ↓
       Wait for Media Processing
                ↓
       Publish Instagram Post
```

## Workflow Stages

### 1. Daily Content Scheduler

`01 - Daily Content Scheduler` starts the workflow at **9:00 AM**. The exported workflow does not explicitly define a timezone, so confirm the n8n workflow/instance timezone before relying on the intended local schedule.

### 2. AI Content Generator

`02 - AI Content Generator` uses `03 - OpenAI Language Model`, configured for **GPT-5 Mini**.

The prompt asks for one Instagram post about AI Automation, aimed at:
- Students
- Developers
- Business owners
- Freelancers

The system instructions also allow topics such as artificial intelligence, technology, digital marketing, UI/UX design, programming, and business.

The generated JSON is expected to contain:

| Field | Purpose |
|---|---|
| `title` | Post title |
| `image_prompt` | Prompt for image generation |
| `caption` | Instagram caption |
| `hashtags` | Array of 5–8 hashtags |

The caption instructions specify a maximum of 35 words, a professional and modern tone, 1–2 relevant emojis, and a simple call to action.

### 3. Pollinations Image Generator

`04 - Pollinations Image Generator` builds a URL from the AI-generated `image_prompt`, URL-encodes the prompt, and requests an image using the `black-forest-labs/flux.1-schnell` model. The HTTP Request node is configured to receive the response as a file.

The node uses a generic HTTP Header Auth credential in the original n8n setup. That credential is removed from the public export and must be reconnected in the user's own n8n instance.

### 4. Upload Image to Cloudinary

`05 - Upload Image to Cloudinary` uploads the generated image file to Cloudinary. The next node uses the resulting `secure_url` as the image's public URL.

Reconnect the Cloudinary credential after importing the public workflow.

### 5. Prepare Instagram Post Data

`06 - Prepare Instagram Post Data` creates two fields:
- `image_url ` — the Cloudinary `secure_url`. The exported field name contains a trailing space.
- `caption` — parsed from the AI Agent's JSON output.

The downstream publish expression also refers to `image_url ` with the trailing space, so the whitespace is consistent in the supplied export. It should still be cleaned up carefully in both places if the workflow is revised.

### 6. Create Instagram Media

`07 - Create Instagram Media` sends a POST request to the Meta Graph API `media` edge, using:
- The Cloudinary image URL
- The prepared caption
- The hard-coded Instagram business account ID in the original export

This step creates the media container used by the later publishing step.

### 7. Wait for Media Processing

`08 - Wait for Media Processing` sits between media-container creation and publishing.

**The exported node has an empty `parameters` object.** No explicit wait duration or resume condition is visible in the supplied JSON. Configure and test the required media-processing wait behaviour before depending on this stage.

### 8. Publish Instagram Post

`09 - Publish Instagram Post` sends a POST request to the Meta Graph API `media_publish` edge using the media container's `creation_id`.

The supplied export also includes `image_url` and `caption` query parameters on this publish request. Verify the request against the current Meta Graph API requirements during setup; the key container reference in the export is `creation_id`.

## What the Current Connections Actually Publish

The AI Agent generates four content fields: `title`, `image_prompt`, `caption`, and `hashtags`.

The downstream `Prepare Instagram Post Data` node passes only:
- The generated image URL
- The generated caption

The `title` and `hashtags` are **not used in the downstream publishing path** in the supplied export. In particular, the generated hashtags are not appended to the caption sent to Instagram. Add them to the caption deliberately if they should appear in the published post.

## Technology Stack

- **n8n** — scheduling and workflow orchestration
- **OpenAI GPT-5 Mini** — Instagram content and image-prompt generation
- **Pollinations** — image generation using `black-forest-labs/flux.1-schnell`
- **Cloudinary** — image hosting and public HTTPS image URL
- **Meta Graph API** — Instagram media-container creation and publishing

## Screenshots

See [screenshots/README.md](./screenshots/README.md) for the supplied workflow overview screenshot.

## Setup

1. Import `workflow.json` into n8n. The public export is inactive.
2. Reconnect OpenAI credentials for `03 - OpenAI Language Model`.
3. Reconnect the HTTP Header Auth credential for `04 - Pollinations Image Generator`, using the authentication expected by the Pollinations endpoint.
4. Reconnect Cloudinary credentials for `05 - Upload Image to Cloudinary`.
5. Reconnect Meta/Facebook Graph API credentials for both Instagram Graph API nodes.
6. Replace `REPLACE_WITH_INSTAGRAM_BUSINESS_ACCOUNT_ID` in both Graph API nodes with the intended Instagram Professional account ID.
7. Confirm that the Instagram account is eligible for publishing through the configured Meta API connection and that the required permissions are granted.
8. Configure the `08 - Wait for Media Processing` node with a suitable wait/polling approach and verify the media container is ready before publishing.
9. Review the publish request parameters against the current Meta Graph API requirements.
10. Decide whether to include the generated hashtags in the final caption. The supplied workflow currently omits them.
11. Confirm the n8n timezone for the 9:00 AM schedule.
12. Test the full workflow with a controlled test post and verify the final post appears on the intended Instagram account before activating the schedule.

## Important Limitations and Production Checks

### 1. Meta Graph API credentials are not included in the export

The supplied JSON contains the Instagram account ID but no credential blocks on the two Facebook Graph API nodes. Reconnect valid Meta credentials and confirm permissions before expecting publishing to work. The supplied n8n screenshot shows warning indicators on both Graph API nodes.

### 2. Instagram account ID is hard-coded

Both Graph API nodes use the same fixed numeric account ID in the source export. The public export replaces it with a placeholder. Use the intended Instagram Professional account ID for both nodes.

### 3. Wait node has no explicit configuration in the export

The `08 - Wait for Media Processing` node has an empty `parameters` object. The screenshot labels it as a media-processing wait, but the JSON does not specify a wait duration or a polling/readiness check. Configure and test this stage.

### 4. Generated hashtags and title are not published

The AI generates `title` and `hashtags`, but the Set node forwards only `image_url ` and `caption`. Hashtags therefore do not reach the publish request. The title is also not used in the downstream path.

### 5. Timezone is not pinned in the workflow export

The scheduler is set to 9:00 AM but no explicit timezone is included. Set the intended timezone and test the next scheduled execution.

### 6. No publishing log or error-handling path is shown

The supplied workflow has no Google Sheets/Database post log, no explicit failure branch, and no notification step for failed image generation, Cloudinary upload, media-container creation, or publishing. Add these if you need an auditable production process.

### 7. Image generation depends on external services

The workflow relies on Pollinations and Cloudinary. Their credentials, availability, response format, and generated media URL must be verified in your own environment.

### 8. AI output is parsed directly

Expressions use `JSON.parse()` on the AI Agent output. If the model returns malformed JSON or missing fields, later nodes may fail. Validate the JSON structure before using its fields in the image generator and publishing steps.

## Current Project Status

The supplied export demonstrates an intended end-to-end sequence for scheduled content generation, image creation, image hosting, media-container creation, and Instagram publishing. It does **not** prove a completed live Instagram post. The exported Meta nodes show warnings in the supplied screenshot, credentials are not included, the media wait node is unconfigured in the JSON, and generated hashtags are not passed into the published caption.

Treat this as a documented prototype until the Meta connection, wait behaviour, final request, and a controlled live test are confirmed.

## Author

**Jasvant Malvi**

- GitHub: https://github.com/jasvantMalvi
- LinkedIn: https://www.linkedin.com/in/jasvant-malvi-810305429/
