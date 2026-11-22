# AI-Powered WhatsApp Quiz Bot

An n8n workflow that delivers a sequential multiple-choice quiz through WhatsApp and uses Google Sheets to store questions, participant progress, and answers.

## Workflow overview

1. A WhatsApp message triggers the workflow.
2. Greeting messages receive instructions to type `START`.
3. `START` retrieves the first question (`Q001`) from the question bank.
4. The workflow sends the question and four options to the participant.
5. A participant's reply is compared with the stored correct answer.
6. The response, correctness, score, and timestamp are appended to the response log.
7. The participant's current question is updated and the next question is sent.
8. After question 10, the workflow calculates a result summary and sends it over WhatsApp.

## Tech stack

- n8n
- WhatsApp Business Cloud API
- Google Sheets
- JavaScript (n8n Code nodes)

## Google Sheets structure

The workflow references three sheets in one spreadsheet:

| Sheet | Purpose |
|---|---|
| `Quiz_Questions` | Stores question IDs, question text, options A–D, and the correct answer |
| `Quiz_Stage` | Stores each participant's phone number, current question ID, and last-question timestamp |
| `Quiz_Responses` | Stores submitted answers, correct answers, correctness, score, and response timestamp |

### Expected columns

**Quiz_Questions**  
`Question_ID`, `Question`, `Option_A`, `Option_B`, `Option_C`, `Option_D`, `Correct_Answer`

**Quiz_Stage**  
`Phone_No`, `Current_Question_ID`, `Last_Question_Time`

**Quiz_Responses**  
`Phone_No`, `Question_ID`, `Student_Answer`, `Correct_Answer`, `Is_Correct`, `Score`, `Response_Time`

## Setup

1. Import `workflows/whatsapp-quiz-bot.json` into n8n.
2. Create a Google Spreadsheet with the three sheet tabs and columns listed above.
3. Add your WhatsApp Trigger and WhatsApp Business credentials in n8n.
4. Connect the Google Sheets credential and select your spreadsheet in the Google Sheets nodes.
5. Set the WhatsApp Phone Number ID in each WhatsApp send node.
6. Add at least 10 question rows with IDs `Q001` through `Q010`.
7. Test with a participant number that can message your WhatsApp Business number.

The exported workflow should have account credential references, webhook IDs, the original spreadsheet ID, and the original WhatsApp Phone Number ID removed before it is shared publicly. Configure your own values after import.

## Current limitations

This is a rule-based quiz workflow; it does not use an AI model to generate questions.

The uploaded workflow currently:
- compares the incoming text with the correct answer;
- does not explicitly reject replies outside A/B/C/D;
- can create duplicate stage rows if a participant sends `START` repeatedly;
- calculates the final result from all response rows for that phone number, so repeat quiz attempts can mix results;
- does not provide a dedicated recovery path when a participant replies without an active quiz-stage row.

These behaviors should be addressed before using the workflow for multiple independent quiz attempts or production use.
