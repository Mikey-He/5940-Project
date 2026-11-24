# Job Application Assistant vision

## What we are building and for whom

Job Application Assistant is a Claude plugin that fills job application forms from a student's confirmed profile. It also learns how to operate difficult form controls from a single demonstration, then shares that fix with other students after it passes tests and review.

Our users are college students and recent graduates applying to many internships and early-career roles. They fill dozens of similar forms each season. They already use Claude and are willing to run Claude Code with Chrome. Two example users guide our decisions.

- **Maya**, a junior applying to 40 summer internships. Her résumé upload keeps putting her minor in the major field, and the university dropdown on many forms rejects pasted text. She wants each form done in minutes and wants to check every value before she submits.
- **Daniel**, a master's student who answers "Why this company?" on most applications. He has good answers saved in a notes file but loses time adapting them to each job description and its word limit.

## The problem

Students retype the same facts on every application, and résumé parsers often place them in the wrong fields. Hard controls are the worst part. A searchable school field can take minutes to fix. So can a date split into three boxes or a repeated "add another education" section. Every student repeats the same fix on the same kind of form, because nothing they learn is kept or shared.

## The role of the LLM

Claude reads field labels in context and decides which profile fact belongs in each field. It drafts answers to open questions from the job description when no saved answer fits. It turns a student's demonstrated fix into general steps. Code does everything that must be the same every time.

- Storing the profile and matching known fields
- Checking rules before they run
- Removing personal values from shared fixes
- Reading values back after a fill

## MVP features in scope

| # | Feature | What the student can do | How we know it works |
| --- | --- | --- | --- |
| 1 | Profile setup | Drag the résumé PDF into Claude Code, or type its file path. A cover letter file is added the same way. Claude extracts contact details plus each degree and job as separate records. The student corrects and confirms each record before any form uses it. | Re-running setup on the same résumé creates no duplicate records. Unconfirmed records are never filled. |
| 2 | Fill an application | Say "fill this application" on an open form. The plugin plans the whole page and fills it in one batch. It uploads the registered résumé or cover letter to file fields. It reads every value back and lists any field it skipped or could not verify. Multipage forms continue page by page. | On our three fixture forms, known fields are filled correctly and every unfillable field is flagged, never guessed. |
| 3 | Sensitive questions | The student sets fixed answers once for sponsorship and work authorization. Demographic questions work the same way. The plugin fills only those exact answers and never drafts or learns them. | No sensitive answer ever comes from a rule or a draft. |
| 4 | Open-question answers | Saved answers are suggested within the scope the student chose for each one. When none fits, Claude drafts one from the job description within the word limit. The student approves or edits it before it is typed. | Drafts respect the word limit. Nothing is typed without approval. |
| 5 | Teach a fix | When a control fails, the student shows Claude the working steps once. The plugin saves a personal rule and uses it next time on the same kind of control. | A taught fix succeeds on the next fixture run for the same student. |
| 6 | Shared fix library | The student may contribute a personal rule. Personal values are replaced by profile references before the student approves the export. A pull request then runs schema and replay tests. After a teammate's review, every install downloads it at its next start. | A second synthetic applicant needs fewer corrections on the same controls. Shared rules contain no personal values. |
| 7 | Review and undo | Every run ends with a list of changed fields and a log. The student can undo the plugin's changes on the current page. | Undo restores the values the page had before the run. |

The student always reviews and submits the application personally.

## Features explicitly out of scope

| Out of scope | Reason |
| --- | --- |
| Automatic submission or signing agreements | The student must stay responsible for what is sent. |
| Logging in or creating accounts, including CAPTCHAs | Credentials stay with the student. Claude in Chrome pauses at these steps. |
| Email tracking and job ranking. Interview preparation too | Each is a separate product. We cut them to finish the core well. |
| Scraping job boards such as LinkedIn | Against their terms and not needed for filling forms. |
| Guaranteed support for every real employer site | The MVP is tested on three fixture forms. Real sites are tried and reported separately. |
| A hosted backend or cloud sync of profiles | Profiles stay on the laptop. Shared rules travel as files in a public folder. |
| Merging shared rules without human review | Every shared rule needs passing tests and a teammate's approval. |
| Non-English forms and mobile browsers | Not needed for the demo. |
| Writing a new cover letter for each job | The student's own cover letter file is uploaded as it is. Tailoring it is a separate feature. |

## Success criteria for the December 16 demo

1. Three synthetic applicants complete our three fixture forms. Together the runs cover at least 60 predefined field decisions, and at least 57 are correct. Every incorrect or unknown field is flagged in the review list.
2. Applicant A teaches two fixes. Applicant B, on the same controls, needs fewer corrections than A did.
3. A scan of every shared rule finds no personal values from any synthetic applicant.
4. When we change a fixture control so a rule no longer fits, the read-back catches it. The rule is then disabled on that laptop and the field is flagged.
5. An open-question draft stays within the word limit and is not typed until the student approves it.

## Constraints and assumptions

- Each student needs a paid Claude plan and Claude Code.
- Each student needs Google Chrome with the Claude in Chrome extension, plus Node.js 22.13 or later.
- The plugin runs locally. The only hosted services are Claude itself and the public GitHub folder that holds shared rules.
- We assume our MCP server and Claude in Chrome work in one Claude Code session. A teammate checks this before Construction. If the check fails, filling falls back to a field-by-field fill sheet shown in chat.
- Claude sees the page text and the profile fields needed for that page, because it types them. The README says this plainly.

## Open questions

- How many demonstrations should a fix need before we offer to share it, one or two?
- Should a shared rule be disabled for everyone after a set number of failed read-backs, or only on the laptop where it failed?
- Which controls do our three fixture forms need so they reflect real application sites? We propose a searchable school field and a split date field. We also propose repeated education sections inside a two-page flow.
- Who reviews contributed rules during the demo period, and how quickly?
- Should saved open-question answers expire after a season?
- Should the plugin keep its own copy of the résumé and cover letter, or only remember where the student keeps them? We propose remembering the path and checking the file is unchanged before each upload.
