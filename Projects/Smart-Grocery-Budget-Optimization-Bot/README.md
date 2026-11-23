# Smart Grocery Budget Optimization Bot

A Telegram-based grocery planning and budget optimization workflow built with n8n, OpenAI GPT-5 Mini, JavaScript, and ElevenLabs Speech-to-Text. It accepts text and voice messages, classifies the user's request into one of five intents, and routes it to a dedicated AI processing branch.

**This is the separate `Smart Grocery Budget Optimization Bot` project. It is not the same project as `Smart Grocery Budget Optimizer Bot`.**

## What It Does

The workflow supports five request types:

1. **Grocery Budget** — suggests a category-wise allocation and practical ways to manage a stated grocery budget.
2. **Grocery List** — estimates the cost of requested items and quantities, optimizes the purchase against the user's budget, and generates a recovery plan when the optimized estimate remains over budget.
3. **Price Check** — returns an estimated price and range for one requested grocery item.
4. **Budget Analysis** — analyses a budget and spending amount explicitly supplied in the user's message.
5. **General Question** — answers a non-grocery question through a separate AI Agent branch.

The bot accepts text messages and voice notes. Voice notes are downloaded from Telegram and transcribed through ElevenLabs before they enter the same intent-classification stage.

> **Price accuracy:** This workflow does not connect to a live grocery catalogue, retailer website, price API, or browser/search tool. Grocery prices are model-generated estimates, not verified live market prices. Check local stores or trusted retailers before making a purchase.

## Workflow Architecture

The supplied export contains **61 nodes**:

| Node type | Count |
|---|---:|
| JavaScript Code | 26 |
| AI Agent | 10 |
| OpenAI Chat Model | 10 |
| Telegram | 8 |
| If | 3 |
| Telegram Trigger | 1 |
| Set / Edit Fields | 1 |
| ElevenLabs | 1 |
| Switch | 1 |

All 10 AI Agents use **GPT-5 Mini** in the supplied export.

### 1. Telegram Input and Voice Transcription

```text
Telegram Trigger
      ↓
Check for text message
      ├── Text → Extract text → Normalize input ─────────┐
      │                                                 │
      └── No text → Check for voice note                │
                      ├── Voice → Download file         │
                      │          → ElevenLabs STT       │
                      │          → Normalize voice text│
                      └── Neither → Send input prompt   │
                                                        ↓
                                              Grocery Intent Parser
```

The text and transcribed voice paths converge before intent classification.

### 2. Intent Classification

The `Grocery Intent Parser` AI Agent is instructed to return JSON containing:

- `intent`
- `message`
- `budget`
- `items`
- `quantity`
- `preferences`
- `constraints`

The `Parse Grocery Intent` Code node parses the response, validates the intent against the five allowed values, normalizes the budget field, and passes the result to the `Switch` node.

### 3. Grocery Budget Branch

```text
Prepare Budget Data
   ↓
Grocery Budget Advisor
   ↓
Parse Grocery Budget Advisor
   ↓
Calculate Budget Allocation
   ↓
Grocery Budget Recommendations
   ↓
Parse Budget Recommendations
   ↓
Format Budget Report
   ↓
Send Budget Report (Telegram)
```

This branch produces recommended grocery spending categories, allocation amounts, remaining budget, and practical recommendations. It does not retrieve actual purchases or save a budget record.

### 4. Grocery List and Optimization Branch

```text
Prepare Grocery Data
   ↓
Build Grocery Item List
   ↓
AI Agent – Grocery Price Researcher
   ↓
Parse Grocery Price Research
   ↓
Calculate Grocery Budget
   ↓
Grocery Budget Optimizer
   ↓
Parse Optimization Result
   ↓
Budget Status
   ├── Optimized cost ≤ budget
   │       ↓
   │   Prepare Final Grocery Report
   │       ↓
   │   Final Grocery Advisor
   │       ↓
   │   Parse Final Grocery Advisor
   │       ↓
   │   Format Telegram Response
   │       ↓
   │   Send Final Grocery Report
   │
   └── Optimized cost > budget
           ↓
       Budget Recovery Optimizer
           ↓
       Parse Budget Recovery Result
           ↓
       Format Recovery Report
           ↓
       Send Final Grocery Report1
```

The optimization prompt instructs the AI to preserve requested quantities by default, prefer lower-cost equivalent products or suppliers, and treat quantity reductions as optional last-resort suggestions. The recovery branch is intended to find further savings when the initial optimized estimate remains above the user's budget.

### 5. Price Check Branch

```text
Prepare Price Check Data
   ↓
Grocery Price Checker
   ↓
Parse Price Check Result
   ↓
Format Price Check Response
   ↓
Send Budget Report1 (Telegram)
```

The AI Agent returns an estimated unit price, unit, estimated price range, and advice. The prompt explicitly tells the agent not to claim a live price unless it has been verified.

### 6. Budget Analysis Branch

```text
Prepare Budget Analysis Data
   ↓
Grocery Spending Analyzer
   ↓
Parse Spending Analysis
   ↓
Format Budget Analysis Report
   ↓
Send Budget Analysis (Telegram)
```

The JavaScript input node extracts a budget and spent amount from the user's message. This is an analysis of the numbers supplied in that message, not an analysis of a stored transaction history.

### 7. General Question Branch

```text
Prepare General Question Data
   ↓
General Question Answerer
   ↓
Format General Question Response
   ↓
Send General Question Response (Telegram)
```

This branch handles messages that do not match the four grocery-specific intents.

## Example Telegram Prompts

| Intent | Example message |
|---|---|
| Grocery Budget | `My monthly grocery budget is ₹20,000.` |
| Grocery List | `My budget is ₹5,000. Rice 20kg, wheat flour 10kg, sugar 5kg and oil 5L.` |
| Price Check | `What is the estimated price of Basmati rice per kg?` |
| Budget Analysis | `My grocery budget is ₹10,000 and I spent ₹7,500 this month.` |
| General Question | `Explain the difference between oats and wheat.` |
| Voice Input | Send a Telegram voice note containing any of the above requests. |

## Technology Stack

- **n8n** — workflow orchestration and routing
- **Telegram Bot API** — text and voice input, report delivery
- **OpenAI GPT-5 Mini** — intent classification, price estimation, budget advice, optimization, and general answers
- **ElevenLabs Speech-to-Text** — voice-note transcription
- **JavaScript** — input normalization, JSON parsing, budget calculations, branch preparation, and Telegram formatting
- **JSON** — structured inputs and AI-agent outputs

## Project Files

- [n8n Workflow JSON](./workflow.json) — sanitized export of the actual 61-node workflow.
- [Screenshots](./screenshots/README.md) — workflow overview and example Telegram outputs.

## Setup

1. Import `workflow.json` into your n8n instance. The published package is intentionally inactive.
2. Configure Telegram credentials on the Telegram Trigger, file download, and Telegram send nodes.
3. Configure OpenAI credentials on the 10 OpenAI Chat Model nodes.
4. Configure ElevenLabs credentials on the transcription node.
5. Confirm that your ElevenLabs plan and node configuration support Speech-to-Text for the Telegram voice files you intend to process.
6. Test the text input branch with one prompt for each of the five intents.
7. Test a Telegram voice note and confirm that its transcription reaches the Grocery Intent Parser.
8. Test both the within-budget and over-budget outcomes of the grocery-list branch.
9. Test malformed or incomplete messages before activating the workflow.
10. Activate the workflow only after credentials, Telegram routing, and test cases have been checked.

## Known Limitations and Issues to Review

The following points are based on the supplied workflow JSON, not assumptions about future changes.

### 1. Grocery prices are not live or verified

The price-research and price-check branches use GPT-5 Mini prompts. The workflow contains no HTTP Request, retailer API, grocery catalogue, web search, or external price source. Returned prices are estimates generated by the model and may be inaccurate or outdated.

### 2. A grocery list without a budget can be routed as a zero-budget request

The intent prompt allows `budget: null` when no budget is provided. However, `Prepare Grocery Data` converts a missing budget to `0`. The later budget check then compares the estimated cost with zero, which can send a list-only request into the over-budget recovery branch. Ask the user for a budget or handle a missing budget explicitly before relying on this path.

### 3. Invalid AI JSON can silently become zero-valued financial data

Several parsing nodes catch JSON errors and continue with empty objects or default numeric values. For example, an invalid price-research response can result in an estimated total of zero; an invalid optimization response can default its optimized cost to zero and its status to `OPTIMIZED`. Add strict schema validation and stop the branch when the AI output is invalid. Do not treat a zero-cost result as a real estimate.

### 4. The no-text / no-voice fallback is missing a recipient chat ID

The `Send a text message` node on the false output of `If1` contains a message body but no `chatId` parameter in the supplied export. The branch may therefore fail when a Telegram update contains neither text nor a voice file. Map the incoming chat ID into that node.

### 5. The budget-analysis example screenshot does not fully match the current JSON mapping

The screenshots show a budget-analysis response with a `Potential Savings` line and written advice. In the supplied export, `Parse Spending Analysis` calculates remaining budget and spending status but does not map the AI output's `potential_savings` or `advice` fields into `potentialSavings` and `spendingAdvice`. The formatter reads those properties. Update the parsing node or verify the actual execution before relying on the screenshot as proof of those two fields.

### 6. Budget analysis is not historical expense tracking

The budget-analysis branch extracts a budget and spent amount from the current message. There is no Google Sheets, database, or transaction ledger node in this workflow. It cannot independently know what the user spent across previous purchases or months.

### 7. No persistence or per-user budget profile

The supplied workflow does not save user budgets, item lists, price results, or spending history. It does not provide a persistent user profile or conversation memory. Each request is processed from the current message and the information passed through that execution.

### 8. Budget-analysis input expects both numbers

The `Parse Spending Analysis` node throws an error if the budget is missing or less than or equal to zero, or if the spending amount is negative. The input-extraction code initializes unmatched budget and spending values to zero, so vague messages may fail rather than request clarification.

### 9. Quantity and price assumptions still need user review

The AI price researcher is instructed to estimate a reasonable retail price and explain unit assumptions. It is not connected to a verified seller or stock check. Quantities and substitutions should be reviewed before a real purchase.

### 10. No dedicated error-handling branches

The supplied workflow does not show dedicated fallback paths for Telegram delivery failures, OpenAI failures, ElevenLabs transcription failures, rate limits, or invalid AI output. Add error handling and monitoring before using it as a public-facing service.

## Current Project Status

The repository package documents a multi-intent Telegram grocery assistant with text and voice input, budget allocation, grocery-list optimization, budget recovery, estimated price checks, spending analysis, and general question answering.

It is a substantial prototype, but **it is not connected to live grocery prices and does not store a user's actual spending history**. The known parsing, missing-budget, and fallback-chat-ID issues should be addressed before production use.

## Author

**Jasvant Malvi**

- GitHub: https://github.com/jasvantMalvi
- LinkedIn: https://www.linkedin.com/in/jasvant-malvi-810305429/
