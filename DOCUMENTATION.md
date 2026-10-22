# AI Candidate Screening Agent — Documentation

> Source document text converted to Markdown for repository viewing. Layout and some table formatting from the original DOCX may differ.

AI Candidate Screening Agent
   Recruitment screening automation · n8n workflow · Human approval
   21 October 2026

   Optimity Logics | contact@example.com


The AI scores the resume, a person makes the decision

 14                          50                             4                             0
 nodes in the workflow       minimum score for              weighted scoring              candidates rejected by
                             recruiter review               factors                       the AI



1. At a glance
 Item                       Detail

 Task                       AI Candidate Screening Agent (Recruitment)

 What it does               Takes a job application with a PDF resume, compares it with the job in a Google
                            Sheet, scores the match with Google Gemini, saves the report and sends strong
                            candidates to a recruiter for approval

 Built with                 n8n, Google Gemini, Google Sheets, Gmail

 Core idea                  The AI gives decision support only. It never rejects anyone, and a candidate is told
                            they are shortlisted only after a recruiter clicks Approve


2. The business problem
A single job post can bring many applications. Recruiters have to open each resume, compare it with the
job description, note the strengths and gaps, and record the result. This takes time, and different people
judge the same resume differently.
The manual process today:
  1. Receive the application and the resume.
  2. Read the resume and the job description side by side.
  3. List the skills, experience and projects that match.
  4. Note what is missing or not shown.
  5. Give a rough score and record it in a sheet.
  6. Decide who to shortlist and email them.

3. The solution
An n8n workflow that starts from an application form. The resume text is extracted, the job details are
read from the Jobs sheet, and Google Gemini compares the two using fixed fairness rules. The workflow
then calculates a transparent weighted score. Scores of 50 or more are saved for review and sent to the
recruiter by email with Approve and Decline buttons. Lower scores are only logged for further review.
The candidate is emailed only after approval.




            The workflow in n8n: screening on the top row, review and decision on the bottom row.


4. How it works
 Part         Node                               What it does

 Screen       Candidate Application Form         Collects full name, email, phone and a PDF resume. A hidden
                                                 job ID is filled from the application link

 Screen       Extract Resume Text from PDF       Turns the PDF into text

 Screen       Get Job Details                    Finds the job in the Jobs sheet by job ID: title, description and
                                                 required experience

 Screen       Prepare Screening Prompt           Stops with an error if the job is missing, not open, or the
                                                 resume has no text. Builds the AI prompt with fairness rules

 Screen       Message a model                    Google Gemini returns the candidate profile, matching skills,
                                                 missing items and four scores as JSON

 Screen       Parse Analysis and Calculate       Calculates the weighted score, the review category and an
              Score                              application ID, and prepares the report fields

 Review       Shortlist Check (Score >= 50)      Sends scores of 50 or more to the review path and lower
                                                 scores to the log path

 Part             Node                                What it does

 Review           Save to Further Review Tab          Logs lower scores. No rejection and no email

 Review           Save to Shortlisted Tab             Adds the report with the status Pending

 Review           Recruiter Approval (Send and        Emails the recruiter the full report and pauses until Approve
                  Wait)                               or Decline is clicked

 Decide           Recruiter Approved?                 Splits the flow into approved and declined

 Decide           Email Candidate - Shortlisted       Sends the shortlist email to the address the candidate typed
                                                      in the form

 Decide           Mark Shortlisted in Sheet           Sets the status to Shortlisted, Candidate Notified to Yes and
                                                      adds the decision date

 Decide           Mark Not Shortlisted in Sheet       Sets the status to Not Shortlisted. No email is sent


5. How candidates are scored
Gemini gives four scores from 0 to 100, based only on evidence in the resume. The workflow then
combines them with fixed weights, so the final score is easy to explain.

 Factor                         Weight            What it measures

 Skills                         50%               Skills in the resume that match the job requirements

 Experience                     25%               Relevant experience compared with the required experience

 Projects                       15%               Projects that show the needed skills

 Certifications                 10%               Only when the job asks for them. Otherwise the weight is shared
                                                  across the other three


 Overall score                  Review category                    What happens

 75 to 100                      Priority Review                    Saved and sent to the recruiter

 50 to 74                       Standard Review                    Saved and sent to the recruiter

 Below 50                       Further Review                     Logged only, no email


6. Rules the AI follows
   • It looks only at job-related evidence in the resume.
   • It must not use or guess age, gender, race, religion, disability, nationality, photo, family status or
     any other protected characteristic.
   • It must not make a hiring decision or reject a candidate.
   • It must not invent facts. If the resume does not show a requirement, it writes 'not demonstrated'
     instead of saying the candidate lacks the skill.

   • It returns JSON only, so the workflow can calculate the score.
   • Scores are described as decision-support estimates, not a final recommendation.


7. Data saved to Google Sheets
 Sheet                 Content                                                Notes

 Jobs                  Job ID, job title, job description, required           Read for every application. Only jobs
                       experience and status                                  with status Open are accepted

 Shortlisted           One row per candidate with 26 columns, status          Updated after the recruiter decides
                       starts as Pending

 Further Review        The same 26 columns for scores below 50                Logged only


The report columns are: application ID, candidate name, job title, overall match score, review priority,
recruiter review status, email, phone, job ID, submitted at, skills, experience, projects and certifications
match, required and candidate experience, candidate skills, matching skills, missing or not demonstrated
requirements, relevant projects, AI screening summary, recruiter notes, job description, resume file
name, candidate notified and decision date.


8. Tools and technologies
 Tool                       Used for                                   Why

 n8n                        Form, flow, scoring code and approval      One place for the whole process

 Google Gemini              Reading the resume and scoring the         Returns structured JSON
                            evidence

 Google Sheets              Jobs, reports and decisions                Simple for recruiters to filter and review

 Gmail                      Recruiter approval and candidate email     Approve and Decline buttons pause the
                                                                       workflow until a person decides


9. Scenarios the workflow is built to handle
These describe the designed behavior. Test results are not part of this document.

 Scenario                                   Designed behavior

 Strong match (75 or more)                  Report saved, recruiter emailed, Priority Review

 Medium match (50 to 74)                    Report saved, recruiter emailed, Standard Review

 Weak match (below 50)                      Logged under Further Review, no email, no rejection

 Recruiter clicks Approve                   Candidate gets the shortlist email, sheet shows Shortlisted and Notified
                                            Yes

 Recruiter clicks Decline                   Sheet shows Not Shortlisted, no email is sent

Scenario                              Designed behavior

Job not found or not open             The workflow stops with a clear error

Resume has no text, such as a         The workflow stops with an error asking for a text-based PDF
scanned PDF


10. Human approval and safety points
  • The AI never rejects a candidate. Low scores are only logged.
  • The shortlist email goes out only after a recruiter clicks Approve.
  • A declined candidate is marked in the sheet and receives no email.
  • The approval email states that the report is AI decision support and that final decisions stay with a
    human.
  • Fairness rules in the prompt keep the AI to job-related evidence.
  • The candidate's own typed name, email and phone are used instead of AI-extracted ones.


11. Limitations and what production needs
  • This document has no recorded test results. Run the scenarios in section 9 with real resumes first.
  • The report link in the approval email points to a different spreadsheet than the one the workflow
    writes to. It should be checked.
  • Candidates waiting for approval sit in the tab named Shortlisted with the status Pending, which
    can be confusing.
  • Candidates get no confirmation when they apply, and those who are declined or scored below 50
    get no reply.
  • Only PDF resumes with selectable text work. Scanned PDFs need OCR. Only the first 45,000
    characters are read.
  • If the job link is wrong or the job is closed, the workflow stops with an error and the candidate
    sees no helpful message.
  • AI scores can differ between runs for the same resume.
  • The approval step has no reminder or time limit.
  • There is no check for duplicate applications.
  • Automated screening and resume data may be covered by local hiring and privacy rules, which
    should be reviewed.


12. Key learnings and improvements
  • AI scores, rules calculate. Fixed weights make the final score transparent and easy to explain.
  • Evidence only. 'Not demonstrated' is fairer than saying a candidate lacks a skill.

  • Human decision at the end. The approval step keeps responsibility with the recruiter.
  • Next improvements: an application confirmation email, a polite reply to all candidates after
    review, OCR for scanned resumes, duplicate checks, a reminder for pending approvals and clearer
    tab names.

Optimity Logics | Email: contact@example.com
