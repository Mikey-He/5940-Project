# Job Application Assistant vision

## What we are building and for whom

Job Application Assistant is a Claude plugin that covers three parts of a student's job search in one place.

- **Apply.** It fills job application forms from the student's confirmed profile, and it learns how to operate a difficult form control from a single demonstration.
- **Track.** It keeps one record of every application and its status, from ranked to submitted to interview to offer or rejection.
- **Rank.** It scores how well a job posting fits the student from 0 to 100, based on a preference questionnaire, and explains the score with evidence from the posting.

Our users are college students and recent graduates applying to many internships and early-career roles. They fill dozens of similar forms each season, lose track of where they applied, and have no quick, consistent way to decide which postings deserve an application. They already use Claude and are willing to run Claude Code with Chrome. Two example users guide our decisions.

- **Maya**, a junior applying to 40 summer internships. Her résumé upload keeps putting her minor in the major field, and the university dropdown on many forms rejects pasted text. She wants each form done in minutes, wants to check every value before she submits, and wants to know which of her saved postings fit her best.
- **Daniel**, a master's student who answers "Why this company?" on most applications. He has good answers saved in a notes file but loses time adapting them to each job description and its word limit. Replies arrive in his inbox weeks later, and he no longer remembers which applications are still open.

## The problem

Students retype the same facts on every application, and résumé parsers often place them in the wrong fields. Hard controls are the worst part. A searchable school field can take minutes to fix. So can a date split into three boxes or a repeated "add another education" section. After submitting, applications scatter across portals and inboxes, so students lose track of where they applied and who replied. Before applying, they face dozens of postings with no consistent way to judge which ones match their skills, location and pay needs.

## The role of the LLM

Every input in this product is unstructured language, which is why an LLM fits.

- **Apply.** Claude reads field labels in context and decides which profile fact belongs in each field. It drafts answers to open questions from the job description when no saved answer fits. It turns a student's demonstrated fix into general steps.
- **Track.** Claude reads an employer email or portal message and recognizes whether it is a confirmation, interview invitation, rejection or offer, and which application it belongs to.
- **Rank.** Claude extracts location, work arrangement and pay from a posting, and judges skills fit, interest fit and nice-to-have fit with quotes from the posting as evidence.

Code does everything that must be the same every time.

- Storing the profile, the tracker and the rankings, and matching known fields
- Checking rules and hard constraints, and computing the final fit score
- Applying status rules, including computing No response
- Reading values back after a fill, and keeping undo history

## MVP features in scope

| # | Feature | What the student can do | How we know it works |
| --- | --- | --- | --- |
| 1 | Profile setup | Drag the résumé PDF into Claude Code, or type its file path. A cover letter file is added the same way. Claude extracts contact details plus each degree and job as separate records. The student corrects and confirms each section before any form uses it. | Re-running setup on the same résumé creates no duplicate records. Unconfirmed records are never filled. |
| 2 | Fill an application | Say "fill this application" on an open form. The plugin plans the whole page and fills it in one batch. It uploads the registered résumé or cover letter to file fields. It reads every value back and lists any field it skipped or could not verify. Multipage forms continue page by page after the student approves each page. | On our three fixture forms, known fields are filled correctly and every unfillable field is flagged, never guessed. |
| 3 | Sensitive questions | The student sets fixed answers once for sponsorship and work authorization. Demographic questions work the same way. The plugin fills only those exact answers and never drafts or learns them. | No sensitive answer ever comes from a rule or a draft. |
| 4 | Open-question answers | Saved answers are suggested within the scope the student chose for each one. When none fits, Claude drafts one from the job description within the word limit. The student approves or edits it before it is typed. | Drafts respect the word limit. Nothing is typed without approval. |
| 5 | Teach a fix | When a control fails, the student shows Claude the working steps once. The plugin saves a personal rule and uses it next time on the same kind of control. | A taught fix succeeds on the next fixture run for the same student. |
| 6 | Review and undo | Every run ends with a list of changed fields and a log. The student can undo the plugin's changes on the current page. | Undo restores the values the page had before the run. |
| 7 | Preference questionnaire | At setup the student answers a short questionnaire: target roles, industries, locations, work arrangement, minimum pay, company size, start date, must-haves and nice-to-haves, and the No response cutoff. Answers can be edited any time. | Every answer is validated, and ranking refuses to run without a questionnaire. |
| 8 | Application tracker | Each filled application is logged automatically with company, role, link and date. The student can add others by hand. Statuses run Ranked, In progress, Submitted, Interview invited, Interview scheduled, then Offer or Rejected. No response is computed after a set number of days. Only the student confirms Submitted. The tracker shows as a table in chat with filters and a timeline per application, and it exports to CSV. | Logging the same link twice never creates a duplicate. No response appears exactly at the cutoff. |
| 9 | Status updates | The student tells Claude about a reply or pastes an employer message. Claude classifies it and proposes the change. Clear matches apply automatically with undo, and uncertain ones wait for the student. Reading Gmail through Claude's read-only connector is a stretch goal. | Status accuracy is reported on labeled test messages, and every uncertain match goes to review. |
| 10 | Job fit ranking | For a posting on the open page or pasted text, Claude extracts its details and the plugin returns a 0–100 fit score. The score explains what matches, what is missing and any deal-breakers. Hard constraints (location, arrangement, minimum pay) are checked in code. Several saved postings can be compared side by side. | Every hard-constraint violation is flagged, every score cites the posting, and the team agrees with the top-3 ordering. |

The student always reviews and submits the application personally.

## Features explicitly out of scope

| Out of scope | Reason |
| --- | --- |
| Automatic submission or signing agreements | The student must stay responsible for what is sent. |
| Logging in or creating accounts, including CAPTCHAs | Credentials stay with the student. Claude in Chrome pauses at these steps. |
| Sharing fixes across students (shared rule library) | Deferred to keep three features achievable by December 16. Personal fixes still work. |
| Searching or scraping job boards such as LinkedIn | Against their terms. Students bring the postings they want ranked. |
| Continuous background email monitoring, or any Gmail write access | Status checks run when the student asks, and we never send, label or delete mail. |
| Interview preparation | A possible future feature that could use tracker data. |
| Guaranteed support for every real employer site | The MVP is tested on three fixture forms. Real sites are tried and reported separately. |
| A hosted backend or cloud sync of profiles | Profiles, the tracker and rankings stay on the laptop. |
| Non-English forms and mobile browsers | Not needed for the demo. |
| Writing a new cover letter for each job | The student's own cover letter file is uploaded as it is. |

## Success criteria for the December 16 demo

Measured on synthetic data during Construction before any claims are made.

1. **Filling.** Three synthetic applicants complete our three fixture forms. Together the runs cover at least 60 predefined field decisions, and at least 57 are correct. Every incorrect or unknown field is flagged in the review list. Re-running a form creates no duplicate entries.
2. **Learning.** Applicant A teaches two fixes. On a later run of the same controls, applicant A needs fewer corrections than on the first run.
3. **Privacy.** A scan of personal rules, run history and logs finds no profile values. Rules hold profile references only.
4. **Safety.** When we change a fixture control so a rule no longer fits, the read-back catches it. The rule is then disabled and the field is flagged.
5. **Drafts.** An open-question draft stays within the word limit and is not typed until the student approves it.
6. **Tracking.** On at least 20 labeled synthetic employer messages (confirmations, interview invites, rejections, offers, unrelated), we report status accuracy, and every uncertain match goes to review instead of being applied.
7. **Ranking.** At least 10 synthetic postings are ranked against two synthetic questionnaires. Every hard-constraint violation is flagged and every score includes quoted evidence. The team agrees with the top-3 ordering for each questionnaire.

## Constraints and assumptions

- Each student needs a paid Claude plan and Claude Code.
- Each student needs Google Chrome with the Claude in Chrome extension, plus Node.js 22.13 or later.
- The plugin runs locally. The only hosted service is Claude itself, plus Claude's read-only Gmail connector if the student chooses to connect it.
- We assume our MCP server and Claude in Chrome work in one Claude Code session. A teammate checks this before Construction. If the check fails, filling falls back to a field-by-field fill sheet shown in chat.
- Claude sees the page text, pasted messages and postings, and the profile fields needed for the current task. The README says this plainly.

## Open questions

- Which questionnaire questions matter most for ranking, and are the default weights (skills 50%, interest 30%, nice-to-haves 20%) right?
- How do we judge whether a score is right? We propose team agreement on labeled test postings.
- Can every teammate connect the Gmail connector, or do we demo status updates from pasted messages only?
- Which real employer sites should we try, and how do we report what works?
- Should saved open-question answers expire after a season? We decided to mark them "old" and still suggest them.
