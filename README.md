# AI Candidate Screening Agent Using n8n

An AI-assisted recruitment workflow that analyzes PDF resumes, compares candidates with job requirements, calculates transparent match scores, and sends screening reports to recruiters for human review.

## 2. Overview

This n8n workflow collects candidate details and a PDF resume, extracts resume text, retrieves job requirements from Google Sheets, and uses Google Gemini to generate a structured screening report. A weighted score considers skills (50%), experience (25%), projects (15%), and certifications (10%) when applicable; weights are redistributed when a category is not relevant. Candidates meeting the configured score threshold are sent to a recruiter for a human decision. Lower-scoring applications are recorded for further review and are not automatically rejected.

## 3. Demo Video

[Watch the demo on YouTube](https://youtu.be/UyFoETAyois?si=JIBvi3GEheWINVk1)

## 4. Tech Stack & Tools

- n8n workflow automation
- Google Gemini
- Google Sheets
- PDF text extraction
- Gmail recruiter approval and notifications

## 5. Prerequisites

- An n8n instance
- Google Gemini API access and an n8n credential
- Google Sheets OAuth credential and a configured spreadsheet
- Gmail OAuth credential
- Job data and the expected review tabs in Google Sheets (starter CSV templates are in `database/`)
- Text-selectable PDF resumes; scanned PDFs may require OCR

## 6. Setup / Import Workflow JSON into n8n

1. Download `workflow.json` from this repository. It has been sanitized; reconnect credentials and replace spreadsheet placeholders before running.
2. In n8n, select **Workflows → Import from File** and choose the JSON export.
3. Reconnect Google Sheets, Gmail, and Gemini credentials.
4. Replace placeholder spreadsheet and sheet references with your own values.
5. Save and test the workflow with non-production sample data before enabling it.

## 7. Configuration Details

- **Job lookup:** Configure the Google Sheets document and job-data tab used by the workflow.
- **Review tabs:** Prepare the Shortlisted and Further Review tabs with the columns expected by the workflow.
- **Gemini model:** models/gemini-3.5-flash-lite (choose a currently available model in your n8n environment if needed).
- **Score threshold:** 50/100 for recruiter review routing.
- **Recruiter approval:** Configure the Gmail recipient and review the score/report before approving or declining.
- **Privacy:** Use test data during setup. Do not commit credentials, candidate resumes, personal data, spreadsheet IDs, or private links to a public repository.

## 8. Step-by-Step Run

1. Submit the application form with candidate details, a resume PDF, and the relevant job ID.
2. The workflow extracts the resume text and retrieves job requirements from Google Sheets.
3. Gemini produces a structured comparison and the workflow calculates the weighted match score.
4. Scores at or above 50 are logged in Shortlisted with a pending review status and sent to the recruiter for approval.
5. The recruiter makes the final decision through the approval email. A candidate shortlist email is sent only after approval.
6. Scores below 50 are logged in Further Review; the workflow does not automatically reject or email those candidates.

## 9. Sample Input / Output

**Input:** Candidate contact details, a PDF resume, and a job ID.

**Screening report may include:** candidate profile, matching skills, requirements that need verification, component scores, overall match score, and a review category. Scores are decision-support signals, not a substitute for a fair human assessment. Verify all extracted facts and apply consistent, job-related criteria.

No formal test results are included in this repository; run the workflow with representative test cases before production use.

## 10. Documentation

[Open the project documentation](DOCUMENTATION.md)

## 11. Footer

Built by Optimity Logics

#OptimityLogics
