# Client Registration Form

A simple institute course-enrollment workflow built with n8n, an n8n Form Trigger, Google Sheets, and Gmail. A prospective student submits a registration form, n8n inserts or updates the registration record in a Google Sheet, and Gmail sends a formatted welcome email to the submitted email address.

## Workflow Overview

The supplied workflow contains **3 connected nodes**:

```text
On form submission
        ↓
Append or update row in sheet
        ↓
Send a message (Gmail)
```

| Step | Node | Responsibility |
|---|---|---|
| 1 | On form submission | Displays the institute registration form and captures the submitted fields |
| 2 | Append or update row in sheet | Saves the registration details to Google Sheets, matching existing records by Email Address |
| 3 | Send a message | Sends an HTML welcome email to the email address submitted in the form |

## Registration Form Fields

The form title is **Client Registration Form** and its description is:

> Welcome to our Institute, Fill the form to enroll in Courses

| Field | Form type | Available choices / notes |
|---|---|---|
| Full Name | Text | Applicant's full name |
| Contact No. | Number | Contact number |
| Email Address | Email | Used as the matching field in Google Sheets and as the welcome-email recipient |
| Cources | Dropdown | Ai Agents, UI/UX, Digital Marketing, Data Analytics |
| City | Text | Applicant's city |
| Gender | Checkbox | Male, Female |
| Message | Text | Additional message from the applicant |

**Field-name note:** The exported workflow uses the spelling `Cources` in the form and Google Sheets mapping. It is retained in this package to preserve the existing field mapping. If you correct it to `Courses`, update the form field, Google Sheets header, and mapping consistently.

## Google Sheets Behaviour

The Google Sheets node uses the `appendOrUpdate` operation and matches records using the **Email Address** column.

That means:
- A registration with a new email address is appended as a new row.
- A submission using an email address already in the matching column updates that existing row.
- The workflow does not create a separate registration-history row for every repeat submission using the same email address.

The exported column mappings are:

- Full Name
- Contact No.
- Email Address
- Cources
- City
- Gender
- Message

The workflow targets the `Sheet1` tab in the connected spreadsheet.

## Welcome Email

After the Google Sheets node, the Gmail node sends an HTML email with the subject:

**Welcome to Our Institute!**

The email includes:
- A welcome greeting using the submitted full name.
- Confirmation that the registration details were received.
- A registration-details card containing name, email, phone, selected course, city, gender, and message.
- A note that the admissions team will contact the applicant with next steps.
- Institute website, institute name, and address placeholders in the template.

The recipient is taken from the submitted `Email Address` value.

## Screenshots

See [screenshots/README.md](./screenshots/README.md) for the available workflow screenshot.

## Technology Stack

- n8n Form Trigger
- Google Sheets
- Gmail
- HTML email template
- n8n expressions for mapping submitted values

## Setup

1. Import `workflow.json` into your n8n instance. The public export is inactive.
2. Create a Google Spreadsheet with a tab named `Sheet1`.
3. Add these column headers exactly as mapped: `Full Name`, `Contact No.`, `Email Address`, `Cources`, `City`, `Gender`, and `Message`.
4. Replace `REPLACE_WITH_YOUR_GOOGLE_SHEET_ID` in the Google Sheets node with your spreadsheet ID.
5. Connect your Google Sheets OAuth credentials.
6. Connect your Gmail credentials to the `Send a message` node.
7. Review the Gmail HTML template and replace `https://yourwebsite.com`, `Your Institute Name`, and other institute placeholders with your actual details.
8. Submit a test registration using an email address you control.
9. Confirm that the row is inserted in `Sheet1` and that the welcome email arrives at the submitted address.
10. Submit a second test using the same email address and verify that the existing matching row is updated rather than appended.

## Important Limitations and Checks

### 1. Email address is the upsert key

The Google Sheets node matches on `Email Address`. Two different people sharing the same email address would update the same matching row. The workflow has no separate registration ID or duplicate-review step.

### 2. Gender checkbox handling in the email

The form defines Gender as a checkbox with Male and Female options. The HTML email template uses `{{ $json['Gender'][0] }}`, which displays only the first selected value. It does not render every selected value. If no value is selected, the expression may not produce the intended output.

### 3. Contact number uses a number field

The Contact No. field is configured as a number. Phone numbers are identifiers, not quantities; a numeric form field may not preserve leading zeros or formatting. A text field with an appropriate pattern/length validation is safer for phone numbers.

### 4. Form fields are not explicitly marked required in the export

The supplied form configuration does not mark the fields as required. Add required-field validation for Full Name, Contact No., Email Address, and Cources if these are mandatory for enrollment.

### 5. No separate admission approval or course confirmation

The email confirms receipt of the registration only. The workflow does not check course availability, verify payment, approve admission, or enroll the student in a learning platform.

### 6. Placeholder institute details

The HTML email includes `https://yourwebsite.com`, `Your Institute Name`, and generic institute contact/location wording. Replace these placeholders before sending real messages.

### 7. No dedicated error-handling path

The workflow has no alternate route for a Google Sheets failure, a Gmail send failure, or invalid form data. Add an error path and operational alerting if this is used for live registrations.

### 8. Personal data handling

The form collects contact details, email address, city, gender, and an optional message. Restrict access to the spreadsheet and avoid collecting fields that are not needed for the institute's registration process.

## Current Project Status

The supplied workflow is a functional three-step registration automation design: capture applicant details, upsert the record in Google Sheets, and send a welcome email. It does **not** implement admission approval, payment verification, or learning-platform enrollment.

## Author

**Jasvant Malvi**

- GitHub: https://github.com/jasvantMalvi
- LinkedIn: https://www.linkedin.com/in/jasvant-malvi-810305429/
