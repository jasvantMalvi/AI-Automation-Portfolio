# AI Automation Portfolio — Project Directory

<p align="center">
  <img src="https://img.shields.io/badge/Projects-50-2563eb?style=for-the-badge" alt="50 projects">
  <img src="https://img.shields.io/badge/Primary%20Platform-n8n-ea4b71?style=for-the-badge&logo=n8n&logoColor=white" alt="n8n">
  <img src="https://img.shields.io/badge/Focus-AI%20%26%20Automation-0f766e?style=for-the-badge" alt="AI and automation">
</p>

A practical collection of **50 AI automation and agent projects** built around real workflow use cases: customer communication, business operations, data processing, productivity, education, content creation and more.

Most projects use **n8n** to connect AI models with tools such as Google Sheets, Gmail, Telegram, WhatsApp, Airtable, cloud services and external APIs.

> **Portfolio transparency:** This repository contains a mix of tested demonstrations and prototypes. Not every workflow is production-ready. Each project README should be treated as the source of truth for its implemented features, setup requirements and known limitations.

---

## Explore the portfolio

| Project area | Projects |
|---|---:|
| Customer Support & Communication | 10 |
| Business Operations, Finance & HR | 11 |
| Education, Career & Student Workflows | 8 |
| Data, Documents & Research | 4 |
| Marketing, Content & Creative Automation | 5 |
| Personal Productivity & Lifestyle | 8 |
| Grocery Planning & Consumer Utilities | 3 |
| Healthcare Prototype | 1 |
| **Total** | **50** |

## Project directory

Project IDs are kept stable to make it easier to refer to a project across this portfolio. Select a project name to open its folder and documentation.

### Customer Support & Communication

| ID | Project | Overview |
|:--:|---|---|
| 01 | [Enterprise AI WhatsApp Support Chatbot](./Enterprise-AI-WhatsApp-Support-Chatbot/) | AI-powered customer support over WhatsApp, using n8n, OpenAI and Airtable. |
| 02 | [AI IT Help Desk and Incident Resolution Agent](./AI-IT-Help-Desk-Incident-Resolution-Agent/) | Processes IT support requests, classifies issues, assesses priority, suggests troubleshooting and supports incident escalation. |
| 03 | [AI KYC Customer Verification](./AI-KYC-Customer-Verification/) | Handles KYC document intake, identity checks, duplicate lookup, risk assessment and manual-review routing. |
| 18 | [AI Email Classifier by Priority Bot](./AI-Email-Classifier-by-Priority-Bot/) | Monitors Gmail, classifies messages by category and priority, creates summaries and sends priority-based WhatsApp notifications. |
| 22 | [New Enquiry Notifier Bot](./New-Enquiry-Notifier-Bot/) | Captures form enquiries, creates unique enquiry IDs, validates details, alerts the business owner on Telegram and emails the customer. |
| 27 | [AI Email Generation Bot](./AI-Email-Generation-Bot/) | Generates professional email subjects and bodies from form inputs, stores the results in Google Sheets and sends them through Gmail. |
| 32 | [Lead Follow-up Automation Bot](./Lead-Followup-Automation-Bot/) | Tracks new and returning WhatsApp leads, detects course interest, updates Google Sheets and sends welcome or course-interest replies. |
| 38 | [AI Voice Telegram Assistant](./AI-Voice-Telegram-Assistant/) | Transcribes Telegram voice messages, generates conversational responses with memory, converts replies to speech and returns audio. |
| 41 | [Telegram AI Assistant](./Telegram-AI-Assistant/) | A text-based Telegram chatbot that answers general questions and also sends a copy of each response to a configured Gmail inbox. |
| 46 | [AI Outlook Auto Reply Agent](./AI-Outlook-Auto-Reply-Agent/) | Checks Outlook for unread emails, classifies them, drafts structured replies, logs them in Sheets, sends replies and marks messages as read. |

### Business Operations, Finance & HR

| ID | Project | Overview |
|:--:|---|---|
| 04 | [AI Invoice Analyzer Bot](./AI-Invoice-Analyzer-Bot/) | Processes invoice emails, extracts and validates invoice details with AI, records results in Google Sheets and generates an email report. |
| 07 | [Monthly Client Work & Billing Automation System](./Monthly-Client-Work-and-Billing-Automation-System/) | Tracks client work logs, calculates billing, generates PDF invoices, maintains report history and delivers reports through Telegram. |
| 10 | [AI Maintenance Dispatch](./AI-Maintenance-Dispatch/) | A four-workflow maintenance prototype for request intake, issue analysis, technician matching and status validation; see its README for integration gaps. |
| 15 | [Smart Business Analyst Bot](./Smart-Business-Analyst-Bot/) | Analyzes weekly sales, calculates KPIs, identifies trends, creates forecasts and risk insights, and delivers management reports with a Looker Studio dashboard. |
| 16 | [Smart Bill Tracker & Alert Bot](./Smart-Bill-Tracker-and-Alert-Bot/) | Tracks bill due dates, identifies upcoming and overdue payments, filters paid bills and manages reminder history. |
| 20 | [AI Attendance Tracking Bot](./AI-Attendance-Tracking-Bot/) | Records student attendance from WhatsApp messages, captures date and time, checks duplicate daily entries and sends confirmations. |
| 21 | [Office Holiday Notifier](./Office-Holiday-Notifier/) | Checks a Google Sheets holiday calendar and sends Gmail announcements when configured notification dates are approaching. |
| 25 | [Smart Expense Tracking Bot](./Smart-Expense-Tracking-Bot/) | Extracts expense details from WhatsApp messages, records transactions in Google Sheets and sends expense summaries. |
| 26 | [Meeting Scheduler Automation](./Meeting-Scheduler-Automation/) | Checks Google Calendar availability for submitted meeting requests, creates available events, updates Sheets and emails confirmations or rejections. |
| 43 | [Client Registration Form](./Client-Registration-Form/) | Captures applicant details through a form, creates or updates a Google Sheets record and sends a personalized welcome email. |
| 45 | [Leave Request Notifier Bot](./Leave-Request-Notifier-Bot/) | Summarizes new employee leave requests, updates their pending status in Google Sheets and emails the manager. |

### Education, Career & Student Workflows

| ID | Project | Overview |
|:--:|---|---|
| 06 | [AI Attendance Intelligence Agent Using Airtable](./AI-Attendance-Intelligence-Agent-Using-Airtable/) | Analyzes student attendance records, classifies attendance risk and produces automated intelligence reports. |
| 11 | [AI-Powered WhatsApp Quiz Bot](./AI-Powered-WhatsApp-Quiz-Bot/) | Delivers sequential multiple-choice questions, checks answers, tracks participant progress and records quiz scores in Google Sheets. |
| 28 | [Skill Gap Analyzer Bot](./Skill-Gap-Analyzer-Bot/) | Compares a PDF resume with a target role, identifies matching and missing skills, calculates a readiness score and creates a learning roadmap. |
| 34 | [Automated Course Feedback Collector Bot](./Automated-Course-Feedback-Collector-Bot/) | Collects text and voice feedback, transcribes audio, analyzes sentiment and suggestions, checks duplicates and alerts administrators. |
| 35 | [Automated Certificate Delivery Agent](./Automated-Certificate-Delivery-Agent/) | Processes pending recipients, generates certificates from a Google Docs template, shares the files, updates Sheets and emails recipients. |
| 36 | [AI-Powered Batch Schedule Management & Student Notification Agent](./AI-Powered-Batch-Schedule-Management-and-Student-Notification-Agent/) | Processes text or voice schedule-change requests, checks scheduling conflicts, updates batch details and prepares student notifications. |
| 39 | [AI Study Planner Bot](./AI-Study-Planner-Bot/) | Turns Telegram study requests into personalized plans with timetables, weekly subject priorities, revision and exam preparation. |
| 44 | [LinkedIn Job Alert Agent](./LinkedIn-Job-Alert-Agent/) | Reads job records from an existing Apify dataset, creates AI summaries, saves opportunities to Sheets and emails a daily digest. |

### Data, Documents & Research

| ID | Project | Overview |
|:--:|---|---|
| 05 | [AI Competitor Intelligence Agent](./AI-Competitor-Intelligence-Agent/) | Analyzes competitor reviews and feedback, performs sentiment and SWOT analysis, and prepares comparison reports. |
| 08 | [AI Document Summarizer Agent](./AI-Document-Summarizer-Agent/) | Extracts text from documents and generates AI-powered summaries for downstream sharing and review. |
| 19 | [Smart File Organizer Bot](./Smart-File-Organizer-Bot/) | Classifies newly uploaded Google Drive files and routes them into designated folders based on file type. |
| 33 | [Daily News Digest Agent](./Daily-News-Digest-Agent/) | Collects stories from five RSS topic feeds, filters and ranks articles, analyzes trends and produces a daily Google Docs digest. |

### Marketing, Content & Creative Automation

| ID | Project | Overview |
|:--:|---|---|
| 13 | [Social Media Content Reminder Bot](./Social-Media-Content-Reminder-Bot/) | Checks pending social content on a schedule, tracks reminder counts and lets users mark items complete through Telegram. |
| 24 | [Multi Language Translator Bot](./Multi-Language-Translator-Bot/) | Detects a source language, translates text into 20 configured languages and uploads a structured translation report to Google Drive. |
| 29 | [AI Image Generator Bot](./AI-Image-Generator-Bot/) | Uses Telegram prompts to create social post copy and square AI images; the documented workflow uploads images to Google Drive. |
| 37 | [AI YouTube Shorts Generator Bot](./AI-YouTube-Shorts-Generator-Bot/) | Turns TechCrunch RSS articles into structured 30–60 second Shorts script drafts and saves scripts and source details to Google Sheets. |
| 50 | [Instagram Post Automation](./Instagram-Post-Automation/) | Generates Instagram copy and an image prompt, creates an image, hosts it on Cloudinary and submits an Instagram media container for publishing. |

### Personal Productivity & Lifestyle

| ID | Project | Overview |
|:--:|---|---|
| 09 | [AI Powered Personal Assistant in Telegram](./AI-Powered-Personal-Assistant-in-Telegram/) | A text-and-voice personal assistant with integrations for Gmail, Notion tasks, Google Calendar and Google Contacts. |
| 12 | [AI Smart To-Do List Automation](./AI-Smart-To-Do-List-Automation/) | A Telegram task manager for adding, viewing, completing and deleting tasks, with Google Sheets storage. |
| 17 | [Sleep Schedule Reminder Agent](./Sleep-Schedule-Reminder-Agent/) | Uses stored sleep schedules to send personalized bedtime and morning reminders and track reminder delivery. |
| 23 | [AI Astrologist or Pandit](./AI-Astrologist-or-Pandit/) | A WhatsApp astrology assistant that collects birth details and generates Vedic astrology-style responses in Hindi, English or Hinglish. |
| 31 | [AI Powered Tasks Generator Bot](./AI-Powered-Tasks-Generator-Bot/) | Converts Telegram goals into structured task plans, saves task records to Google Sheets and sends a completion summary. |
| 40 | [AI Confidence Coach Agent](./AI-Confidence-Coach-Agent/) | Onboards users, stores their confidence goals and challenges, and provides practical personalized coaching responses. |
| 48 | [Birthday Reminder Automation Bot](./Birthday-Reminder-Automation-Bot/) | Finds active contacts whose birthday matches the current date, generates a personalized WhatsApp wish and updates the contact record. |
| 49 | [Daily Motivation Message Sender Bot](./Daily-Motivation-Message-Sender-Bot/) | Generates a practical motivational message, sends it to a configured Telegram chat and appends a message log to Google Sheets. |

### Grocery Planning & Consumer Utilities

| ID | Project | Overview |
|:--:|---|---|
| 14 | [Smart Grocery Budget Optimizer Bot](./Smart-Grocery-Budget-Optimizer-Bot/) | Estimates grocery shopping costs, compares spending with a stated budget, suggests lower-cost alternatives and logs requests. |
| 30 | [Grocery Restock Reminder Agent](./Grocery-Restock-Reminder-Agent/) | Checks purchase history and restock cycles, identifies likely replenishment needs and sends personalized Telegram reminders. |
| 42 | [Smart Grocery Budget Optimization Bot](./Smart-Grocery-Budget-Optimization-Bot/) | A separate, more extensive grocery-planning assistant for text and voice requests, price estimates, budget allocation, shopping-list optimization and budget-recovery reports. |

### Healthcare Prototype

| ID | Project | Overview |
|:--:|---|---|
| 47 | [Advanced Medical Assistance AI Agent with Emergency Detection and Health Assessment](./Advanced-Medical-Assistance-AI-Agent/) | Collects patient-reported information, checks selected emergency phrases, generates a preliminary AI assessment and stores emergency or standard cases. Not clinically validated and not suitable for real-world triage. |


---

## Tools & technologies

The projects use different combinations of the following tools; **not every project uses every tool**.

- **Workflow automation:** n8n, webhooks, scheduled triggers, JavaScript
- **AI models:** OpenAI, Google Gemini, Anthropic Claude
- **Google Workspace:** Google Sheets, Google Drive, Google Docs, Google Calendar, Gmail
- **Messaging:** Telegram, WhatsApp Business Cloud API, Outlook
- **Data & integrations:** Airtable, Apify, Cloudinary, REST APIs
- **Content generation:** AI text generation, image generation, structured output parsing, document creation

## Repository conventions

Most project folders are organized around the following files, where available:

- `README.md` — purpose, functionality, architecture, setup and known limitations
- `workflow.json` — exported n8n workflow with private credentials removed or replaced
- `screenshots/` — workflow diagrams, sample outputs and supporting evidence

Check each project folder for its actual contents; older or smaller projects may not include every item.

## Notes before reuse

1. Reconnect your own credentials and replace placeholder IDs before importing a workflow.
2. Review each README's limitations before running a workflow with real users or business data.
3. Check external API permissions, usage limits, privacy requirements and timezone settings.
4. Healthcare-related automation in this repository is a prototype and is **not** a substitute for professional medical assessment or emergency services.

### Portfolio inventory note

The current `Projects/` directory contains **50 project folders**. The earlier master list also mentioned **Auto Presentation Generator Agent**, but there is no matching project folder in the repository at present, so it is not counted or linked as an uploaded project here.

---

<p align="center">
  <b>Built by Jasvant Malvi</b><br>
  <a href="https://github.com/jasvantMalvi">GitHub</a> ·
  <a href="https://www.linkedin.com/in/jasvant-malvi-810305429/">LinkedIn</a>
</p>
