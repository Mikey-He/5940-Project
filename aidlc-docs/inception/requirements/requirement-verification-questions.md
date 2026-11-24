# Requirements Verification Questions

These questions come from reading `README.md`, `vision.md` and `tech-env.md`. They cover only what those documents leave open or ambiguous. Decisions already made there (local-only MCP server, `node:sqlite`, Zod, the five rule step types, the prohibited libraries, the three fixture forms and three synthetic applicants, the student always submitting) are treated as fixed and are not asked again.

**How to answer:** put the letter after each `[Answer]:` tag. If no option fits, choose the last option (Other) and describe your choice after the tag. Mikey He answers these for the team in this Inception run. Requirements Analysis does not move on until every question is answered.

---

## Part 1 - Open questions from vision.md

## Question 1
How many demonstrations should a fix need before the plugin offers to share it with other students?

A) One demonstration. The rule must still pass its next read-back on the same student's run before the share option appears

B) One demonstration. The share option appears right after the rule is saved

C) Two demonstrations on two different forms with the same kind of control

X) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 2
When a shared (approved) rule fails a read-back, who should stop using it?

A) Only the laptop where it failed. The team withdraws it for everyone by hand, through a pull request

B) Only that laptop, and the plugin also offers to file a GitHub issue with the failure (field label, fingerprint and step, no personal values)

C) Everyone, automatically, after a set number of reported failures (this needs a hosted failure counter, which conflicts with the "no backend" rule)

X) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 3
Which hard controls must the three fixture forms include?

A) The proposed set only: a searchable school field, a date split into three boxes, and repeated "add another education" sections inside a two-page flow

B) The proposed set plus a file upload field and a yes/no radio group for sponsorship

C) The proposed set plus a file upload field, a sponsorship radio group, a custom (non-native) dropdown, and an open question with a word limit

X) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 4
Who reviews contributed rules during the demo period, and how fast?

A) Any teammate other than the rule's author, within 48 hours

B) A named rule owner from the "Rule learning and validation" stream, within 48 hours

C) Two approvals from teammates other than the author

X) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 5
Should saved open-question answers expire?

A) Never expire. The student deletes them by hand

B) Answers older than one application season (about six months) are still suggested but marked "old, please review"

C) Answers older than one season are hidden until the student renews them

X) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 6
How should the plugin keep the résumé and cover letter?

A) Remember the path, size and SHA-256 hash only, and check the file is unchanged before each upload (the proposal in vision.md and tech-env.md)

B) Keep a private copy in the plugin data folder

X) Other (please describe after [Answer]: tag below)

[Answer]: 

---

## Part 2 - Functional gaps

## Question 7
On a multipage form, who moves to the next page?

A) The student clicks "Next" and then says "continue". The plugin never clicks navigation buttons

B) The plugin clicks "Next" only after the student has reviewed the current page and approved moving on

C) The plugin clicks "Next" automatically when every field on the page was filled and verified

X) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 8
For a field that code could not match (an `ask_claude` field), what may Claude do?

A) Fill it when the label clearly maps to one confirmed profile fact, and mark it "decided by Claude" in the review list. Leave it blank and flagged when unsure

B) Never fill it. List it in the review list with Claude's suggested value for the student to accept

C) Fill it only for text fields. Leave selects, comboboxes, radios and checkboxes blank and flagged unless a rule covers them

X) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 9
How should the student confirm profile records extracted from the résumé?

A) One record at a time (contact details, then each degree, then each job), each confirmed or edited before the next

B) The whole profile shown as one list, with the student editing any item and confirming once

C) Grouped by section (contact, education, experience), confirming each section

X) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 10
How many profiles and résumé versions does one install support?

A) One profile, one résumé and one cover letter

B) One profile with several named résumé and cover letter versions (for example "SWE" and "Data"), chosen per application

C) Several complete profiles

X) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 11
Which sensitive question categories must have fixed answers, and what happens when no answer is set?

A) Sponsorship, work authorization, and voluntary demographic questions (gender, race and ethnicity, veteran status, disability). With no answer set, leave the field blank and flag it

B) The same categories, but with no answer set, choose "I prefer not to answer" when the form offers it, otherwise leave blank and flag

C) Sponsorship and work authorization only. Demographic questions are always left for the student

X) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 12
What does undo cover?

A) The current page only, restoring the values the page had before the run (as in vision.md)

B) Every page of the current run that is still reachable in the browser

X) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 13
Which document formats are accepted?

A) PDF only, for both profile extraction and upload

B) PDF and DOCX for both extraction and upload (tech-env.md accepts both for `register_document`)

C) PDF for extraction. PDF or DOCX for upload

X) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 14
Should the student be able to delete their data from inside the plugin?

A) Yes. One command deletes the profile, answers, personal rules and run history, after a confirmation

B) Yes, per category (profile, answers, rules, history)

C) No. Uninstalling the plugin without keeping data is enough for the MVP

X) Other (please describe after [Answer]: tag below)

[Answer]: 

---

## Part 3 - Non-functional requirements

## Question 15
What time target should one fixture form meet, from "fill this application" to the review list, excluding time the student spends reviewing or approving drafts?

A) Under 2 minutes per page

B) Under 5 minutes for a whole two-page form

C) No hard target. We record the timing and report it

X) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 16
Which operating systems must the plugin run on for the team and the demo?

A) Windows and macOS

B) Windows, macOS and Linux

C) macOS only for the demo, with best effort elsewhere

X) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 17
What unit test coverage should the MCP server meet?

A) No number. Every deterministic decision listed in tech-env.md (matching, planning, rule schema, personal-data scan, answer scope) has unit tests

B) At least 80% line coverage on `server/src`, enforced in CI

C) At least 70% line coverage on `server/src`, enforced in CI

X) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 18
How should the personal-data scan treat a value that might be personal but is not in any synthetic profile, such as a city name or a company name in a step?

A) Reject any literal value in a step. Only profile references and fixed option labels from the page are allowed

B) Reject values matching email, phone, street address, date of birth and ID-number patterns, plus every value in the synthetic profiles. Allow other literals

X) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 19
What should happen when GitHub cannot be reached at startup?

A) Use the cached approved rules silently

B) Use the cached rules and tell the student once that shared rules may be out of date

C) Run with personal rules only until the next successful sync

X) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 20
When should the Chrome plus MCP compatibility spike run, and who decides on the fallback?

A) Before Application Design, so the result shapes the design. The spike owner reports to the team, and the team decides together

B) Before Construction, as README.md says. The spike owner decides

X) Other (please describe after [Answer]: tag below)

[Answer]: 

---

## Part 4 - AI-DLC extension opt-ins

## Question 21: Security Extensions
Should security extension rules be enforced for this project?

A) Yes - enforce all SECURITY rules as blocking constraints (recommended for production-grade applications)

B) No - skip all SECURITY rules (suitable for PoCs, prototypes, and experimental projects)

X) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 22: Resiliency Extensions
Should the resiliency baseline be applied to this project?

**What this extension is.** Enabling it applies a set of directional, design-time best practices for building resilient systems, derived from the AWS Well-Architected Framework (Reliability Pillar) and resilience-review guidance. It steers requirements, design, and code toward fault tolerance, high availability, observability, and recoverability, covering 15 practice areas across business goals, change management, observability, high availability, disaster recovery, and continuous improvement.

**What this extension is NOT.** Enabling it does not make your workload production-ready, nor does it certify or guarantee any availability, RTO, or RPO target. It is a starting point that scaffolds good resiliency decisions early. It is not a substitute for a formal AWS Well-Architected Review of the built system.

A) Yes - apply the resiliency baseline as directional best practices and design-time guidance (recommended for business-critical workloads, as an informed starting point that you can validate and harden before go-live)

B) No - skip the resiliency baseline (suitable for PoCs, prototypes, and experimental projects where rapid iteration matters more than reliability)

X) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 23: Property-Based Testing Extension
Should property-based testing (PBT) rules be enforced for this project?

A) Yes - enforce all PBT rules as blocking constraints (recommended for projects with business logic, data transformations, serialization, or stateful components)

B) Partial - enforce PBT rules only for pure functions and serialization round-trips (suitable for projects with limited algorithmic complexity)

C) No - skip all PBT rules (suitable for simple CRUD applications, UI-only projects, or thin integration layers with no significant business logic)

X) Other (please describe after [Answer]: tag below)

[Answer]: 
