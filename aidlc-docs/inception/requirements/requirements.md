# Requirements - Job Application Assistant

## Change Log

| Version | Date | Change | Source |
| --- | --- | --- | --- |
| 1 | 2026-11-24 | Initial requirements for apply (fill, sensitive questions, open questions, teach a fix, shared fix library, review and undo) | `requirement-verification-questions.md` Q1-Q23 |
| 2 | 2026-11-25 | Added application tracking (FR-11, FR-12), job fit ranking (FR-13) and the preference questionnaire (FR-10). Removed the shared fix library (FR-6) to keep scope achievable. Updated success criteria to SC1-SC7 | `requirement-change-questions.md` Q1-Q15 |

## Intent Analysis Summary

| Item | Assessment |
| --- | --- |
| User request | Build the Job Application Assistant described in `README.md`, `vision.md` and `tech-env.md` using AI-DLC v1.0.1; the team then decided the final product has three features: apply, track and rank |
| Request type | New Project (greenfield) |
| Scope estimate | System-wide: Claude Code plugin manifest, five skills, a local TypeScript MCP server with a SQLite store, fixture forms, synthetic applicants, messages and postings, CI |
| Complexity estimate | Complex: agentic LLM workflows, personal data handling, rule learning with sanitization, status rules, fit scoring, measurable demo criteria |
| Requirements depth | Comprehensive |
| Milestone scope | Inception only for the mid-semester milestone; Construction for every unit by December 16, 2026 |

Sources: `vision.md` (features 1-10, success criteria SC1-SC7), `tech-env.md`, `requirement-verification-questions.md` (Q1-Q23) and `requirement-change-questions.md` (CQ1-CQ15).

## Product Summary

Job Application Assistant is a Claude Code plugin that helps students **apply, track and rank**. Claude (Claude Sonnet 5.5 in development) reads employer pages through Claude in Chrome, reads pasted employer messages and job postings, and a local MCP server makes every deterministic decision. The plugin fills applications from a confirmed profile and learns fixes for hard controls from one demonstration; it keeps one tracker of every application and its status; and it scores how well each posting fits the student from 0 to 100 with evidence. The student always reviews and submits each application personally.

**Users**: college students and recent graduates applying to many internships and early-career roles (Maya, a junior applying to 40 internships; Daniel, a master's student who reuses answers and loses track of replies).

## Division of Responsibility (LLM vs Code)

| Claude (LLM) decides | Code (MCP server) decides |
| --- | --- |
| Reading page structure and field labels in context | Storing and validating profile records, answers, rules, questionnaire, jobs and applications |
| Mapping a field to a profile fact when code cannot match it exactly | Exact field matching and rule attachment |
| Extracting profile records from a résumé | Deduplicating records and enforcing confirmation status |
| Drafting an open-question answer within a word limit | Checking word limits and answer scope |
| Describing a demonstrated fix as general steps | Validating rule schema, replacing personal values with references, personal-data scan |
| Classifying an employer message and matching it to an application, with a confidence and quote | Applying status rules, auto-apply versus pending decisions, No response, undo |
| Extracting posting facts and judging skills, interest and nice-to-have fit with quotes | Hard-constraint checks, the deal-breaker cap, weighting and the final score |
| Talking to the student | Comparing read-back values with the plan, disabling failing rules |

---

## Functional Requirements

### FR-1 Profile Setup (vision 1)
- **FR-1.1** The student provides a résumé by dragging the PDF into the Claude Code prompt or typing its file path; the setup skill always mentions typing the path as the fallback.
- **FR-1.2** Profile extraction accepts PDF only. Uploads accept PDF or DOCX up to 10 MB. (Q13)
- **FR-1.3** `register_document` stores each document's path, size and SHA-256 hash, never a copy. `get_document` confirms the file exists and is unchanged before every upload; otherwise Claude asks for the file again. (Q6)
- **FR-1.4** Claude extracts contact details and each degree and job as separate records; they are validated, deduplicated and stored as unconfirmed.
- **FR-1.5** The student reviews and confirms the profile section by section (contact, education, experience). (Q9)
- **FR-1.6** Unconfirmed records are never used to fill a form.
- **FR-1.7** Re-running setup on the same résumé creates no duplicate records.
- **FR-1.8** One install supports one profile, one résumé and one cover letter. (Q10)

### FR-2 Fill an Application (vision 2)
- **FR-2.1** When the student says "fill this application", Claude reads the page structure once and lists every field with its label, type, options and element reference.
- **FR-2.2** `plan_fill` matches fields to confirmed profile facts and saved personal rules (by control fingerprint) and returns one planned action per field.
- **FR-2.3** For an `ask_claude` field, Claude fills it only when the label clearly maps to exactly one confirmed profile fact, marked "decided by Claude"; otherwise it leaves the field blank and flagged. (Q8)
- **FR-2.4** Claude fills the page in one batch, reads every value back and reports through `record_fill_result`.
- **FR-2.5** For a file field, Claude calls `get_document` and uploads the registered file.
- **FR-2.6** On multipage forms, the plugin clicks "Next" only after the student approves the current page. (Q7)
- **FR-2.7** The plugin never clicks a submit control, never enters a password, never creates an account, and pauses at login pages and CAPTCHAs.
- **FR-2.8** Every unfillable or unverifiable field is flagged, never guessed. Re-running a form creates no duplicate entries.
- **FR-2.9** If Claude in Chrome and the MCP server cannot run in one session, the fill skill falls back to a field-by-field fill sheet in chat. (Q20)
- **FR-2.10** After a fill, the application is logged in the tracker (FR-11.1).

### FR-3 Sensitive Questions (vision 3)
- **FR-3.1** Sensitive categories: sponsorship, work authorization, and voluntary demographic questions (gender, race and ethnicity, veteran status, disability). (Q11)
- **FR-3.2** The student sets a fixed answer once per category, stored in a separate table.
- **FR-3.3** A sensitive field is filled only by exact question match; with no answer set it is left blank and flagged; the plugin never selects "I prefer not to answer" on its own.
- **FR-3.4** Sensitive answers are never drafted, inferred, learned into a rule, or used in ranking.

### FR-4 Open-Question Answers (vision 4)
- **FR-4.1** `find_answers` returns saved answers that match the question within their scope (application, company, general) and fit the word limit.
- **FR-4.2** Answers older than one application season are still suggested but marked "old, please review". (Q5)
- **FR-4.3** When no saved answer fits, Claude drafts one from the job description within the word limit.
- **FR-4.4** No answer is typed until the student approves or edits it; approved answers can be saved with a scope.
- **FR-4.5** Page text and job descriptions are treated as data; instructions inside them are ignored.

### FR-5 Teach a Fix (vision 5)
- **FR-5.1** When a control fails, the student demonstrates the working steps once; `propose_rule` turns them into a personal rule. (Q1)
- **FR-5.2** A rule has a control fingerprint and steps of exactly five types: `type`, `select_option`, `click`, `wait_for`, `read_back`.
- **FR-5.3** Values in steps are profile references or option labels read from the page; no other literal is allowed. (Q18)
- **FR-5.4** A personal rule is used next time on a control with the same fingerprint.
- **FR-5.5** When a rule fails a read-back, it is disabled and the field is flagged. (Q2, adapted: rules are personal only)
- **FR-5.6** Rules stay on the student's laptop.

### FR-6 Shared Fix Library - REMOVED (CQ14)
Sharing fixes across students through a public rule folder, CI review and sync is out of scope for the MVP. Requirements formerly numbered FR-6.1 to FR-6.7 are withdrawn; the rule-failure behavior is kept as FR-5.5.

### FR-7 Review and Undo (vision 6)
- **FR-7.1** Every run ends with a review list of filled, rule-filled, Claude-decided, flagged and skipped fields, plus a run log.
- **FR-7.2** Undo restores the values the current page had before the run. (Q12)
- **FR-7.3** The student always reviews and submits the application personally.

### FR-8 Data Control
- **FR-8.1** One command deletes the profile, answers, sensitive answers, questionnaire, rules, run history, jobs, rankings and the tracker after an explicit confirmation. (Q14)
- **FR-8.2** Uninstalling the plugin deletes its data folder unless the student chooses to keep it.

### FR-9 Fixtures and Test Data
- **FR-9.1** Three static fixture forms include a searchable school field, a three-box date, repeated education sections in a two-page flow, a file upload, a sponsorship radio group, a custom dropdown, and a word-limited open question, plus a changed-control variant. (Q3)
- **FR-9.2** Three synthetic applicants cover at least 60 predefined field decisions.
- **FR-9.3** At least 20 labeled synthetic employer messages (confirmations, interview invites, rejections, offers, unrelated). (CQ13)
- **FR-9.4** At least 10 synthetic postings and two synthetic questionnaires, with team-agreed top-3 orderings. (CQ13)
- **FR-9.5** Only synthetic data is committed.

### FR-10 Preference Questionnaire (vision 7)
- **FR-10.1** At setup the student answers: target roles, industries, locations, work arrangements (remote, hybrid, on-site), minimum pay, company size, start date, must-haves and nice-to-haves. (CQ9)
- **FR-10.2** The questionnaire also holds the score weights (default skills 50%, interest 30%, nice-to-haves 20%; must add up to 100%) and the No response cutoff (default 21 days). (CQ4, CQ11)
- **FR-10.3** Answers are validated and editable at any time; ranking refuses to run without a questionnaire.

### FR-11 Application Tracker (vision 8)
- **FR-11.1** Every application the plugin fills is logged automatically with company, role, link and date; the student can also add an application by hand. Logging the same link again updates the existing entry. (CQ1)
- **FR-11.2** Statuses form a fixed ordered set: Ranked, In progress, Submitted, Interview invited, Interview scheduled, Offer or Rejected. (CQ2)
- **FR-11.3** No response is computed, never stored: a Submitted application with no later update after the cutoff is shown as No response. (CQ4)
- **FR-11.4** Only the student can move an application to Submitted, by confirming it. (CQ3)
- **FR-11.5** A status never moves backwards automatically. (CQ12)
- **FR-11.6** The tracker shows as a table in chat sorted by date applied, with status filters and a timeline per application, and exports to a CSV file. (CQ7)

### FR-12 Status Updates (vision 9)
- **FR-12.1** Status updates come from the student's statement in chat or from an employer email or portal message the student pastes; Claude classifies it (confirmation, interview invite, interview scheduled, rejection, offer, unrelated), matches it to an application, and gives a confidence and an evidence quote. (CQ5)
- **FR-12.2** A proposed update is applied automatically only when it is high confidence, matches exactly one application, and does not conflict with an earlier student edit; it is recorded with an undo entry. Anything else becomes a pending proposal the student approves or rejects. (CQ6)
- **FR-12.3** Recording the same message twice changes nothing.
- **FR-12.4** The student can undo the latest status change or tracker edit.
- **FR-12.5** Stretch goal: with the student's request, Claude reads employer replies through Claude's read-only Gmail connector and proposes updates the same way. The plugin never sends, labels or deletes mail. (CQ5)

### FR-13 Job Fit Ranking (vision 10)
- **FR-13.1** A posting on the open page or pasted posting text can be saved as a job; no duplicates by link; no job-board searching or scraping. (CQ8)
- **FR-13.2** Claude extracts location, work arrangement and the top of the pay range exactly as the posting states them (or unknown), and judges skills fit, interest fit and nice-to-have fit from 0 to 100, each with quotes from the posting. (CQ10)
- **FR-13.3** Code checks hard constraints (location, arrangement, minimum pay). A violation is a deal-breaker that caps the score at 35; a fact the posting does not state is listed as unknown, not a violation. (CQ10)
- **FR-13.4** Code combines the judgements with the questionnaire weights into a 0-100 score and returns what matches, what is missing, deal-breakers and unknowns. (CQ10, CQ11)
- **FR-13.5** Ranking a job creates its tracker entry with status Ranked; several saved jobs can be compared side by side, highest first. (CQ12)
- **FR-13.6** A judgement without evidence from the posting is rejected.

---

## Non-Functional Requirements

### NFR-1 Performance
- **NFR-1.1** A two-page fixture form completes in under 5 minutes from "fill this application" to the review list, excluding the student's review time. (Q15)
- **NFR-1.2** Each run logs timings (tool names, durations, outcomes) for the demo report.

### NFR-2 Security and Privacy (Security Baseline enabled, Q21)
- **NFR-2.1** All personal data (profile, answers, questionnaire, rules, history, jobs, rankings, tracker) is stored only on the student's laptop; no hosted backend; the MCP server makes no network requests and opens no ports.
- **NFR-2.2** Data at rest relies on the operating system's full-disk encryption and a data folder readable only by the student's account. (SECURITY-01, adapted to a local store)
- **NFR-2.3** Every MCP tool validates input with Zod schemas with type, length and size bounds; all SQLite access uses parameterized statements. (SECURITY-05)
- **NFR-2.4** Logs are structured (timestamp, run ID, level, event) and never contain profile values, answers, message text, posting text or document contents. (SECURITY-03)
- **NFR-2.5** No API keys or secrets exist in the product. `.gitignore` excludes `node_modules/`, `*.db`, `.env` and exported CSV files.
- **NFR-2.6** Page text, résumé text, job postings and employer messages are data, not instructions; they can never add a tool call the student did not ask for. (SECURITY-11 misuse case: prompt injection)
- **NFR-2.7** Rules are interpreted as the five fixed step types only; `eval` and `new Function` are prohibited. (SECURITY-13)
- **NFR-2.8** Dependencies are pinned with a committed lock file, installed with `npm ci`, and checked with `npm audit` in CI; CI uses pinned versions. (SECURITY-10)
- **NFR-2.9** Every database and file call has explicit error handling; on error the operation stops (fail closed), and the server has a top-level handler that returns a generic message. (SECURITY-15)
- **NFR-2.10** Gmail, when used, is read only through Claude's connector.

### NFR-3 Reliability
- **NFR-3.1** Tracking and ranking work without Claude in Chrome, so they keep working if the browser path fails.
- **NFR-3.2** A read-back mismatch always disables the responsible rule and flags the field (SC4).
- **NFR-3.3** Every automatic status change can be undone.
- **NFR-3.4** Resiliency Baseline extension is not enabled. (Q22)

### NFR-4 Compatibility
- **NFR-4.1** Windows and macOS, with Google Chrome and Claude in Chrome, Node.js 22.13 or later, and Claude Code on a paid Claude plan. (Q16)
- **NFR-4.2** One bundled server file using only the built-in `node:sqlite` module.
- **NFR-4.3** English-language desktop forms, messages and postings only.

### NFR-5 Testability and Maintainability
- **NFR-5.1** Every deterministic decision (field matching, planning, rule schema, personal-data scan, answer scope, word limits, status rules, No response, auto-apply versus pending, undo, hard constraints, scoring, CSV export) has Vitest unit tests. (Q17)
- **NFR-5.2** Line coverage on `server/src` is at least 70%, enforced in CI. (Q17)
- **NFR-5.3** Property-based tests with fast-check cover round-trips (record and rule serialization, reference substitution, CSV export and re-import) and invariants (one action per field, no personal literal in rules, drafts within word limits, scores within 0-100, any deal-breaker caps at 35, statuses never move backwards automatically). PBT Partial mode: PBT-02, 03, 07, 08, 09 blocking. (Q23)
- **NFR-5.4** Playwright replay tests run taught rules against the fixture forms for all three applicants in CI.
- **NFR-5.5** `claude plugin validate --strict` passes.

### NFR-6 Usability
- **NFR-6.1** All workflows are conversational inside Claude Code; every fill ends with a review list, every score with an explanation, and every status change shows its evidence.
- **NFR-6.2** Error messages are plain and generic, with a suggested next step.

---

## Constraints and Prohibited Choices (from tech-env.md)
- Claude Code plugin format; no `bin/` folder; no name starting with `claude-`.
- TypeScript 5.x, ES modules, `@modelcontextprotocol/sdk` (stdio), Zod 3, `node:sqlite`, esbuild, npm.
- Prohibited: native database modules, Puppeteer or Selenium inside the plugin, LLM SDKs inside the plugin, `axios` or `node-fetch`, job-board scraping libraries, `eval` or `new Function` on rule content, Gmail write access or a custom mail integration.

## Out of Scope (MVP)
Automatic submission or signing; logging in, account creation or CAPTCHAs; sharing fixes across students (CQ14); searching or scraping job boards; continuous background email monitoring and any Gmail write access; interview preparation; guaranteed support for every real employer site; hosted backend or cloud sync; non-English forms and mobile browsers; per-job cover letters; multiple profiles or résumé versions (Q10).

## Success Criteria Traceability (December 16 demo)

| Success criterion (vision.md) | Requirements |
| --- | --- |
| SC1 Filling: 60+ field decisions, at least 57 correct, all wrong or unknown flagged, no duplicates on re-run | FR-2.2 to FR-2.8, FR-9.1, FR-9.2, NFR-1.1 |
| SC2 Learning: applicant A needs fewer corrections on a later run of controls A taught | FR-5.1 to FR-5.4 |
| SC3 Privacy: no profile values in rules, history or logs | FR-5.3, NFR-2.4, NFR-5.3 |
| SC4 Safety: changed control caught by read-back, rule disabled, field flagged | FR-2.4, FR-5.5, NFR-3.2 |
| SC5 Drafts: within word limit and not typed before approval | FR-4.3, FR-4.4 |
| SC6 Tracking: status accuracy on 20+ labeled messages; every uncertain match goes to review | FR-11, FR-12.1 to FR-12.4, FR-9.3 |
| SC7 Ranking: 10+ postings, two questionnaires, every violation flagged, every score with evidence, team agrees with top-3 | FR-10, FR-13, FR-9.4 |

## Open Items Carried Forward
- The Chrome plus MCP spike runs before Construction; the team decides on the fallback (Q20).
- Gmail connector availability for every teammate decides whether FR-12.5 is demoed (CQ5).
- Default score weights are validated against the team's top-3 orderings during Construction.

## Extension Compliance Summary (Requirements Analysis, version 2)

| Rule | Status | Note |
| --- | --- | --- |
| SECURITY-01 | Compliant (adapted) | NFR-2.2 local store; no network transfer of personal data by the server |
| SECURITY-02, 04, 06, 07, 12, 14 | N/A | No intermediaries, HTML endpoints, cloud IAM, network configuration, authentication or deployed monitoring |
| SECURITY-03 | Compliant (adapted) | NFR-2.4 |
| SECURITY-05 | Compliant | NFR-2.3 |
| SECURITY-08 | N/A at this stage | Single local user; revisited in Application Design |
| SECURITY-09 | Compliant | No default credentials; generic errors |
| SECURITY-10 | Compliant | NFR-2.8 |
| SECURITY-11 | Compliant | Misuse cases: prompt injection through pages, postings and messages (NFR-2.6); submit and Submitted-status boundaries (FR-2.7, FR-11.4) |
| SECURITY-13 | Compliant | Safe rule interpretation (NFR-2.7); no downloaded code or rules after removing the shared library |
| SECURITY-15 | Compliant | NFR-2.9 |
| PBT-09 | Compliant | fast-check with Vitest (NFR-5.3) |
| PBT-02, 03, 07, 08 | Planned | Enforced in Functional Design and Code Generation |
