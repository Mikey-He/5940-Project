# Requirement Change Questions - Application Tracking and Job Ranking

**Change request**: The team decided the final product has three features: **apply** (fill applications, already specified), **track** (one record of every application and its status) and **rank** (a fit score for a job posting). Tracking and ranking were previously out of scope. These questions define the two new features. The team's V3 drafts of `vision.md` and `tech-env.md` were used as reference for the options below.

**Not changed by this request**: every requirement already approved in `requirements.md` (profile setup, filling, sensitive questions, open-question answers, teach a fix, shared fix library, review and undo, data control) stays unless a question below says otherwise.

**How to answer**: put the letter after each `[Answer]:` tag. If no option fits, choose the last option (Other) and describe your choice.

---

## Part A - Application Tracking

## Question 1
Which applications go into the tracker?

A) Every application the plugin fills is logged automatically (company, role, link, date), and the student can also add an application by hand that was filled without the plugin

B) Only applications the plugin fills

C) Only applications the student adds by hand

X) Other (please describe after [Answer]: tag below)

[Answer]: A
Rationale: the tracker must hold every application to be useful, and logging automatically on fill removes data entry for the common case.

## Question 2
Which statuses does an application have?

A) A fixed ordered set: Ranked, In progress, Submitted, Interview invited, Interview scheduled, Offer or Rejected, plus No response (computed, never stored)

B) A short set: Applied, Interviewing, Offer, Rejected

X) Other (please describe after [Answer]: tag below)

[Answer]: A
Rationale: the ordered set covers the whole journey from ranking to an outcome, and computing No response in code keeps it consistent.

## Question 3
Who can mark an application as Submitted?

A) Only the student, by confirming it; the plugin never infers submission (it never clicks submit)

B) The plugin, when the page shows a confirmation message

X) Other (please describe after [Answer]: tag below)

[Answer]: A
Rationale: consistent with the rule that the plugin never submits; only the student knows the application was really sent.

## Question 4
When does a Submitted application count as No response?

A) After 21 days with no later update by default; the student can change the number of days in the preference questionnaire

B) After a fixed 21 days

C) After a fixed 30 days

X) Other (please describe after [Answer]: tag below)

[Answer]: A
Rationale: 21 days is a reasonable default, and making it editable answers the V3 open question without extra complexity.

## Question 5
How do statuses get updated?

A) The student tells Claude in chat ("I got rejected from Stripe") or pastes an employer email or portal message; Claude classifies it and proposes the change. Reading Gmail through Claude's read-only Gmail connector is a stretch goal, used only when the student asks, if connector setup works for the team

B) Same as A, with Gmail required in the MVP

C) Only by the student editing statuses directly

X) Other (please describe after [Answer]: tag below)

[Answer]: A
Rationale: classifying pasted employer messages gives the same LLM value as Gmail and can be tested on labeled synthetic messages; Gmail stays optional so connector setup cannot block the demo.

## Question 6
When is a proposed status change applied without asking?

A) Only when Claude rates it high confidence, it matches exactly one application, and it does not conflict with an earlier student edit; it is then applied with an undo entry. Anything else becomes a pending proposal the student approves or rejects. The same message recorded twice changes nothing

B) Never; every change waits for the student's approval

X) Other (please describe after [Answer]: tag below)

[Answer]: A
Rationale: clear updates save the student time, every automatic change can be undone, and anything uncertain waits for the student.

## Question 7
How does the student see the tracker?

A) As a table in the Claude Code chat, sorted by date applied, with status filters and a timeline per application, plus export to a CSV file

B) As a table in chat only

C) As a separate local web page

X) Other (please describe after [Answer]: tag below)

[Answer]: A
Rationale: stays inside Claude Code with no extra UI to build, and CSV export lets the student keep the tracker elsewhere.

---

## Part B - Job Ranking

## Question 8
Which postings can be ranked?

A) A posting on the page the student has open, or posting text the student pastes; each is saved as a job (no duplicates by link). No job-board searching or scraping

B) Postings found by searching job boards

X) Other (please describe after [Answer]: tag below)

[Answer]: A
Rationale: avoids job-board terms-of-service problems and matches how students already find postings.

## Question 9
What does the student provide so a posting can be ranked?

A) A short preference questionnaire at setup: target roles, industries, locations, work arrangement (remote, hybrid, on-site), minimum pay, company size, start date, must-haves and nice-to-haves; editable any time

B) Nothing extra; ranking uses only the résumé profile

X) Other (please describe after [Answer]: tag below)

[Answer]: A
Rationale: ranking needs the student's preferences and hard constraints, which a résumé does not contain.

## Question 10
How is the 0-100 fit score computed?

A) Claude extracts location, arrangement and pay from the posting and judges skills fit, interest fit and nice-to-have fit (each 0-100 with quotes from the posting as evidence). Code checks the hard constraints (location, arrangement, minimum pay): a violation is a deal-breaker that caps the score at 35, and a fact the posting does not state is listed as unknown, not a violation. Code combines the three judgements with weights

B) Claude gives one overall score with an explanation

X) Other (please describe after [Answer]: tag below)

[Answer]: A
Rationale: the LLM judges what needs language understanding, with evidence; code makes the deal-breaker and weighting decisions, which keeps scores explainable and testable.

## Question 11
How are the score weights set?

A) Defaults of skills 50%, interest 30%, nice-to-haves 20%; the student may change them in the questionnaire, and code checks they add up to 100%

B) Fixed defaults only

X) Other (please describe after [Answer]: tag below)

[Answer]: A
Rationale: sensible defaults with optional student control; the sum check is a simple, testable rule.

## Question 12
How do ranking and tracking connect?

A) Ranking a saved job creates its tracker entry with status Ranked; filling it later moves it to In progress; a status never moves backwards automatically. Several saved jobs can be compared side by side, highest score first

B) Ranking and tracking are separate lists

X) Other (please describe after [Answer]: tag below)

[Answer]: A
Rationale: one record follows a job from ranking to outcome, and never moving backwards automatically protects the student's edits.

---

## Part C - Success Criteria and Scope

## Question 13
What demo success criteria should the two new features add?

A) Tracking: at least 20 labeled synthetic employer messages (confirmations, interview invites, rejections, offers, unrelated); report status accuracy; every uncertain match goes to review instead of being applied. Ranking: at least 10 synthetic postings ranked against two synthetic questionnaires; every hard-constraint violation flagged; every score includes quoted evidence; the team agrees with the top-3 ordering for each questionnaire

B) Tracking and ranking are shown in the demo without measured targets

X) Other (please describe after [Answer]: tag below)

[Answer]: A
Rationale: gives each new feature a measurable target like the filling criteria, taken from the team's V3 draft.

## Question 14
With two features added, which existing MVP features stay in scope for December 16?

A) Keep all existing features (including drafting open-question answers and the shared fix library) and add tracking and ranking; Gmail stays a stretch goal (Question 5) to protect the schedule

B) Drop the shared fix library (keep personal fixes), as in the V3 draft

C) Drop both the shared fix library and drafting of new open-question answers, as in the V3 draft

X) Other (please describe after [Answer]: tag below)

[Answer]: B
Rationale: three features in three weeks is ambitious; deferring the shared library (GitHub index, CI replay, review process) keeps the plan achievable while personal fixes still show learning from one demonstration. Drafting open-question answers stays because it is small and already designed. Gmail stays a stretch goal.

## Question 15
How should the input documents change?

A) Update `vision.md`, `tech-env.md` and `README.md` to add tracking and ranking (features, LLM rationale, MCP tools, data model, success criteria, out-of-scope table), keeping every other approved decision; the V3 drafts are used as reference only

B) Replace `vision.md` and `tech-env.md` with the V3 drafts

X) Other (please describe after [Answer]: tag below)

[Answer]: A
Rationale: keeps the documents consistent with the decisions already approved in this workflow and adds only what the team decided; replacing them wholesale would silently undo other approved decisions.
