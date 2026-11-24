# Advanced Medical Assistance AI Agent with Emergency Detection and Health Assessment

An n8n prototype for collecting patient-reported health information, checking a dedicated emergency-symptoms field for selected warning phrases, generating a structured preliminary AI assessment, and storing the result in separate Google Sheets tabs for emergency and non-emergency cases.

**This is not a diagnostic or clinical decision-support system. It must not be used to determine that a person is medically safe, delay emergency care, or replace a qualified clinician.** The workflow's own form states that it does not provide a medical diagnosis or replace professional care and instructs users in India to call 112 for emergencies.

## Workflow Overview

The supplied export contains **15 nodes**:
- 10 functional nodes, including the form trigger, code, IF nodes, AI chain, model, parser, and two Google Sheets nodes.
- 5 sticky-note annotations.

```text
Patient Health Assessment Form
              ↓
Validate & Normalize Patient Data
              ↓
Initial Emergency Detection
              ↓
AI Medical Assessment Chain
      ↙                    ↘
OpenAI Assessment      Medical Assessment
Model                  JSON Parser
              ↓
Normalize AI Assessment
              ↓
Final Emergency Classification
         ↙                 ↘
Save Emergency Case    Save Non-Emergency Case
(Google Sheets tab)    (Google Sheets tab)
```

Both branches append the assessment to the configured spreadsheet. The workflow does not send a separate notification to emergency services, a doctor, or a hospital.

## Workflow Stages

### 1. Patient Health Assessment Form

`01 - Patient Health Assessment Form` collects:
- Patient name, age, gender, mobile number, and email.
- Main symptoms and symptom duration.
- Pain level from 0 to 10.
- Existing medical conditions, current medications, and allergies.
- A separate Emergency Symptoms field.
- Additional information.
- Required consent checkbox for processing the health information.

The form description says the tool is for preliminary AI-assisted assessment and emergency identification, not diagnosis, and instructs users in India to call 112 for emergencies.

### 3. Validate & Normalize Patient Data

`02 - Validate & Normalize Patient Data` uses JavaScript to:
- Generate a case ID in the `MED-<timestamp>` format.
- Normalize age and pain level to numbers.
- Preserve patient and symptom details.
- Split the Emergency Symptoms field on commas.
- Set absent medical history, medication, and allergy values to `None reported`.
- Record a Boolean `consent_given` field.
- Set `processing_status` to `Ready for AI Assessment`.

The output includes `has_emergency_red_flag`, calculated from the separate Emergency Symptoms field.

### 4. Initial Emergency Detection

`03 - Initial Emergency Detection` checks the `has_emergency_red_flag` Boolean.

**Exported connection issue:** the IF node has only its first (true) output connected to `04 - AI Medical Assessment Chain`. The false output has no downstream connection in the supplied JSON. As exported, submissions without a matching initial red-flag phrase stop at this IF node; they do not reach the AI assessment or the Standard Cases sheet. This must be fixed before the workflow can process non-emergency submissions.

The code compares each comma-separated item in **Emergency Symptoms** against these phrases:
- `severe chest pain`
- `difficulty breathing`
- `unconsciousness`
- `severe bleeding`
- `stroke symptoms`
- `severe allergic reaction`
- `seizure`

This is a simple phrase-matching check, not a comprehensive medical triage assessment.

### 5. AI Medical Assessment Chain

`04 - AI Medical Assessment Chain` sends the normalized patient record to **GPT-5 Mini** through `05 - OpenAI Assessment Model`.

The prompt asks the model to:
- Summarize reported symptoms.
- Identify possible health concerns without diagnosing.
- Classify urgency as Emergency, Urgent, or Standard.
- Explain the reason for the urgency classification.
- Recommend safe next steps.
- Require doctor review for every case.

The prompt also tells the model not to downgrade an existing emergency flag, not to prescribe medication or dosage, not to claim that the patient is medically safe, and to set the status to `Pending Medical Review`. For an emergency, the prompt specifies immediate emergency-care instructions and calling 112 in India.

`06 - Medical Assessment JSON Parser` requires these structured fields:

| Field | Purpose |
|---|---|
| `patient_name` | Patient name |
| `case_id` | Case reference |
| `symptom_summary` | Reported symptom summary |
| `urgency_level` | Emergency, Urgent, or Standard |
| `emergency_detected` | Emergency Boolean |
| `emergency_reason` | Reason for the emergency flag |
| `recommended_action` | Suggested next step |
| `doctor_review_required` | Doctor-review flag |
| `assessment_status` | Assessment status |

### 6. Normalize AI Assessment

`07 - Normalize AI Assessment` extracts the parser output, normalizes the urgency, preserves an emergency indication, sets `doctor_review_required` to `true`, and sets `assessment_status` to `Pending Medical Review`.

If the structured result indicates an emergency, the code sets urgency to `Emergency`. Otherwise it accepts `Urgent` or `Standard`; unexpected urgency values default to `Urgent`.

### 7. Final Emergency Classification

`08 - Final Emergency Classification` routes based on the normalized `emergency_detected` Boolean:
- **True:** `09 - Save Emergency Case`
- **False:** `10 - Save Non-Emergency Case`

### 8. Save the Assessment

Both Google Sheets nodes use the `append` operation and share the same spreadsheet:
- `09 - Save Emergency Case` appends to the **Emergency Cases** tab.
- `10 - Save Non-Emergency Case` appends to the **Standard Cases** tab.

The screenshot shows an example emergency case stored with `Urgency Level = Emergency`, `Emergency Detected = TRUE`, `Doctor Review Required = TRUE`, and `Assessment Status = Pending Medical Review`.

## Screenshots

See [screenshots/README.md](./screenshots/README.md) for the actual n8n workflow and the sample Google Sheets result.

## Technology Stack

- **n8n** — form handling, workflow orchestration, conditional routing
- **n8n Form Trigger** — patient information and consent collection
- **JavaScript** — field normalization, case ID generation, phrase matching
- **OpenAI GPT-5 Mini** — preliminary AI-generated symptom summary and urgency assessment
- **Structured Output Parser** — required assessment fields
- **Google Sheets** — separate emergency and standard case logs

## Setup

1. Import `workflow.json` into your n8n instance. The public export is inactive.
2. Create a Google Spreadsheet named `Advanced Medical Assistance Database`.
3. Create two tabs: `Emergency Cases` and `Standard Cases`.
4. Add the columns shown below to both tabs, matching the sheet mappings exactly.
5. Replace `REPLACE_WITH_YOUR_GOOGLE_SHEET_ID` in both Google Sheets nodes.
6. Reconnect Google Sheets credentials in both nodes.
7. Reconnect OpenAI credentials for `05 - OpenAI Assessment Model`.
8. Review the form wording, privacy notice, data retention, access controls, and consent flow before collecting any real health information.
9. Test only with synthetic cases. Check the emergency and standard branches, case IDs, and stored fields.
10. Do not deploy for real patient triage without qualified clinical, privacy, and safety review and an independently validated emergency response process.

### Google Sheets columns

The supplied sheet screenshot shows these columns in the `Emergency Cases` tab:

- `Timestamp`
- `Case ID`
- `Patient Name`
- `Urgency Level`
- `Symptom Summary`
- `Emergency Detected`
- `Emergency Reason`
- `Recommended Action`
- `Doctor Review Required`
- `Assessment Status`

The workflow maps these same assessment fields into both tabs.

## Critical Limitations and Safety Review

### 1. Emergency phrase detection checks only the separate Emergency Symptoms field

The JavaScript emergency detector reads `Emergency Symptoms`, splits it on commas, and searches for a small list of phrases. It does not inspect `Main Symptoms` for the initial red-flag calculation. A person could describe severe symptoms in the main symptom field while leaving Emergency Symptoms blank or using different wording. This may prevent the initial deterministic flag from being set.

### 3. Phrase matching is brittle and incomplete

The check uses literal substring matching against a short keyword list. It does not cover every emergency symptom, spelling variation, language, abbreviation, or clinical presentation. It must not be treated as a safe triage screen.

### 4. The AI model still influences the final routing

The final emergency branch uses the normalized AI assessment's `emergency_detected` Boolean. The code preserves an emergency indication from the model or the urgency value, and the prompt says not to downgrade an existing red flag. However, the workflow has no separate clinical validation layer to verify model output.

### 5. Emergency cases are stored but not actively escalated

The emergency branch appends a row to Google Sheets. There is no node that contacts 112, calls a hospital, pages a clinician, sends an emergency SMS, or otherwise guarantees that a person sees the record promptly. A spreadsheet entry is not an emergency response.

### 6. Consent is captured but not used as a processing gate

The normalization code records `consent_given`, but the supplied workflow does not include an IF condition that stops processing when consent is false. The form marks the checkbox required, but the workflow itself does not independently enforce the consent field before AI processing or storage.

### 7. Doctor review remains pending for every case

The AI instructions require doctor review, and normalization sets `doctor_review_required` to `true` and `assessment_status` to `Pending Medical Review`. The workflow does not implement a doctor review task, approval interface, or status update.

### 8. No validation for pain range or clinical plausibility

The form asks for pain level from 0 to 10, but the JavaScript code converts the field to a number without checking that it is within that range. Age is also converted to a number without explicit range validation.

### 9. Sensitive health data requires strong safeguards

The form collects symptoms, medical history, medications, allergies, contact details, and consent. The workflow sends the patient record to an AI model and stores assessment results in Google Sheets. Do not use real patient data without an appropriate privacy, security, retention, access-control, and consent review.

### 10. No explicit error-handling or fallback path

The supplied graph does not show dedicated error handling for form processing, model failure, malformed structured output, or Google Sheets write failures. A failed run may leave a submitted case without a stored assessment.

## Current Project Status

The supplied JSON and screenshot demonstrate a working prototype path that accepts a form submission, creates a preliminary AI assessment, classifies the case, and stores the result in an emergency or standard Google Sheets tab. It is **not clinically validated and not suitable for real-world emergency triage**. The emergency phrase check, missing escalation, consent gate, and clinician-review process are major unresolved requirements.

## Author

<p align="left">
  <b>Jasvant Malvi</b><br>
  <a href="https://github.com/jasvantMalvi">GitHub</a> ·
  <a href="https://www.linkedin.com/in/jasvant-malvi-810305429/">LinkedIn</a>
</p>
