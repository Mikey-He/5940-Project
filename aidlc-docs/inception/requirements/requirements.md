# Requirements - Job Application Assistant

## Intent Analysis Summary

| Item | Assessment |
| --- | --- |
| User request | Start a new project: build the Job Application Assistant described in `README.md`, `vision.md` and `tech-env.md` using AI-DLC v1.0.1 |
| Request type | New Project (greenfield) |
| Scope estimate | System-wide: Claude Code plugin manifest, three skills, a local TypeScript MCP server with a SQLite store, a shared rule library with CI replay tests, three fixture forms and three synthetic applicants |
| Complexity estimate | Complex: an agentic LLM workflow, personal data handling, rule learning with sanitization, and measurable demo criteria |
| Requirements depth | Comprehensive: five-person team, personal data, and success criteria that need traceability |
| Milestone scope | Inception only for the mid-semester milestone; Construction for every unit by December 16, 2026 |

Sources: `vision.md` (features F1-F7, success criteria SC1-SC5), `tech-env.md` (architecture, tools, security rules), and the team's answers in `requirement-verification-questions.md` (Q1-Q23).

## Product Summary

Job Application Assistant is a Claude Code plugin that fills job application forms from a student's confirmed profile. Claude (the host LLM, Claude Sonnet 5.5 in development) reads the employer's page through Claude in Chrome, a local MCP server makes every deterministic decision, and the student reviews and submits each application personally. The plugin learns reusable fixes for difficult form controls from one demonstration and shares approved fixes through a public, reviewed rule folder in the team repository.

**Users**: college students and recent graduates applying to many internships and early-career roles (example users: Maya, a junior applying to 40 internships; Daniel, a master's student who reuses open-question answers).

## Division of Responsibility (LLM vs Code)

| Claude (LLM) decides | Code (MCP server) decides |
| --- | --- |
| Reading page structure and field labels in context | Storing and validating profile records, answers and rules |
| Mapping a field to a profile fact when code cannot match it exactly | Exact field matching against known labels and approved rules |
| Extracting profile records from a résumé | Deduplicating records and enforcing confirmation status |
| Drafting an open-question answer within a word limit | Checking word limits and answer scope |
| Describing a demonstrated fix as general steps | Validating rule schema, replacing personal values with references, personal-data scan |
| Talking to the student | Comparing read-back values with the plan, disabling failing rules |

---

## Functional Requirements

### FR-1 Profile Setup (vision F1)

- **FR-1.1** The student provides a résumé by dragging the PDF into the Claude Code prompt or typing its file path. The setup skill always mentions typing the path as the fallback. (tech-env)
- **FR-1.2** Profile extraction accepts PDF only. Uploads accept PDF or DOCX up to 10 MB. (Q13)
- **FR-1.3** `register_document` stores each document's path, size and SHA-256 hash, never a copy. `get_document` confirms the file exists and is unchanged before every upload; otherwise Claude asks the student for the file again. (Q6)
- **FR-1.4** Claude extracts contact details and each degree and each job as separate records; `save_profile_records` validates and deduplicates them and stores them as unconfirmed.
- **FR-1.5** The student reviews and confirms the profile section by section (contact, education, experience). Confirming a section marks every record in it as confirmed; the student may edit any record before confirming. (Q9)
- **FR-1.6** Unconfirmed records are never used to fill a form.
- **FR-1.7** Re-running setup on the same résumé creates no duplicate records.
- **FR-1.8** One install supports one profile, one résumé and one cover letter. A cover letter is registered the same way as the résumé. (Q10)

### FR-2 Fill an Application (vision F2)

- **FR-2.1** When the student says "fill this application", Claude reads the page structure once and lists every field with its label, type, options and element reference.
- **FR-2.2** `plan_fill` matches fields to confirmed profile facts and approved rules (by page fingerprint) and returns one planned action per field: `fill`, `rule` or `ask_claude`.
- **FR-2.3** For an `ask_claude` field, Claude fills it only when the label clearly maps to exactly one confirmed profile fact, and the review list marks it "decided by Claude". When unsure, Claude leaves it blank and flags it. (Q8)
- **FR-2.4** Claude fills the page in one batch, then reads every value back and reports through `record_fill_result`, which compares values with the plan and flags mismatches.
- **FR-2.5** For a file field, Claude calls `get_document` and uploads the registered file through Claude in Chrome.
- **FR-2.6** On multipage forms, the plugin clicks the page's "Next" control only after the student has reviewed the current page and approved moving on. (Q7)
- **FR-2.7** The plugin never clicks a button whose label or type submits the application, never enters a password, never creates an account, and pauses at login pages and CAPTCHAs for the student.
- **FR-2.8** Every unfillable or unverifiable field is flagged in the review list, never guessed.
- **FR-2.9** If the Chrome plus MCP combination is not available in one session, the fill skill falls back to a field-by-field fill sheet in chat that the student copies into the form. (vision constraints, Q20)

### FR-3 Sensitive Questions (vision F3)

- **FR-3.1** Sensitive categories are sponsorship, work authorization, and voluntary demographic questions (gender, race and ethnicity, veteran status, disability). (Q11)
- **FR-3.2** The student sets a fixed answer once per category; answers are stored in a separate table.
- **FR-3.3** A sensitive field is filled only by exact question match with the student's fixed answer. With no answer set, the field is left blank and flagged; the plugin never selects "I prefer not to answer" on its own.
- **FR-3.4** Sensitive answers are never drafted by Claude, never learned into a rule, and never placed in a shared rule.

### FR-4 Open-Question Answers (vision F4)

- **FR-4.1** `find_answers` returns saved answers that match the question within their scope (one application, one company, or general) and fit the word limit.
- **FR-4.2** Saved answers older than one application season (about six months) are still suggested but marked "old, please review". (Q5)
- **FR-4.3** When no saved answer fits, Claude drafts one from the job description within the word limit.
- **FR-4.4** No answer, saved or drafted, is typed until the student approves or edits it. Approved answers can be saved with a scope through `save_answer`.
- **FR-4.5** Page text and job descriptions are treated as data; instructions inside them are ignored.

### FR-5 Teach a Fix (vision F5)

- **FR-5.1** When a control fails, the student demonstrates the working steps once; `propose_rule` turns them into a personal rule. (Q1)
- **FR-5.2** A rule has a target fingerprint and steps of exactly five types: `type`, `select_option`, `click`, `wait_for`, `read_back`. Unknown step types are rejected.
- **FR-5.3** Values in steps are profile references (for example `{{profile.education[0].school}}`) or option labels read from the page; no other literal value is allowed. (Q18)
- **FR-5.4** A personal rule is used next time on a control with the same fingerprint.

### FR-6 Shared Fix Library (vision F6)

- **FR-6.1** The share option appears only after a personal rule has passed one clean read-back on a later run for the same student. (Q1)
- **FR-6.2** `export_rule_candidate` runs the personal-data scan and writes a JSON file only after the student agrees.
- **FR-6.3** Contributions arrive as pull requests to `rules/approved/`. A GitHub Actions workflow runs schema checks and Playwright replay tests for all three synthetic applicants on the three fixture forms.
- **FR-6.4** A rule merges only when every replayed value reads back correctly and one teammate other than the author approves, within 48 hours of the request. (Q4)
- **FR-6.5** `sync_rules` downloads `rules/approved/index.json` at startup over HTTPS, verifies each rule against the schema and the SHA-256 checksum listed in the index, and caches valid rules.
- **FR-6.6** If GitHub cannot be reached, the plugin uses cached rules and tells the student once that shared rules may be out of date. (Q19)
- **FR-6.7** When a shared rule fails a read-back, it is disabled on that laptop only, the field is flagged, and the plugin offers to open a prefilled GitHub issue with the field label, fingerprint and failing step, never personal values. The team withdraws a rule for everyone by removing it from the index in a pull request. (Q2)

### FR-7 Review and Undo (vision F7)

- **FR-7.1** Every run ends with a review list of changed fields, Claude-decided fields, flagged fields and skipped fields, plus a run log.
- **FR-7.2** Undo restores the values the current page had before the run. Earlier pages are not covered. (Q12)
- **FR-7.3** The student always reviews and submits the application personally.

### FR-8 Data Control

- **FR-8.1** One command deletes the profile, saved answers, sensitive answers, personal rules and run history after an explicit confirmation. (Q14)
- **FR-8.2** Uninstalling the plugin deletes its data folder unless the student chooses to keep it.

### FR-9 Fixtures and Test Data

- **FR-9.1** Three static fixture forms include, across them: a searchable school field, a date split into three boxes, repeated "add another education" sections in a two-page flow, a file upload field, a sponsorship radio group, a custom (non-native) dropdown, and an open question with a word limit. (Q3)
- **FR-9.2** Three synthetic applicants (A, B, C) with synthetic résumés cover at least 60 predefined field decisions across the three forms.
- **FR-9.3** Only synthetic data is committed to the repository.

---

## Non-Functional Requirements

### NFR-1 Performance
- **NFR-1.1** A two-page fixture form completes in under 5 minutes from "fill this application" to the review list, excluding the student's review and approval time. (Q15)
- **NFR-1.2** Each run logs its timings (tool names, durations, outcomes) for the demo report.

### NFR-2 Security and Privacy (Security Baseline enabled, Q21)
- **NFR-2.1** Profiles, answers, rules and history are stored only on the student's laptop in the plugin data folder; there is no hosted backend and no cloud sync.
- **NFR-2.2** Data at rest relies on the operating system's full-disk encryption (BitLocker or FileVault, documented as a setup requirement) and on a data folder readable only by the student's account. Network traffic for shared rules uses HTTPS only. (SECURITY-01, adapted to a local store)
- **NFR-2.3** Every MCP tool validates input with Zod schemas that set types, maximum string lengths and array sizes; all SQLite access uses parameterized statements. (SECURITY-05)
- **NFR-2.4** Logs are structured (timestamp, run ID, level, message) and record tool names, outcomes and timings, never profile values, answers or document contents. (SECURITY-03)
- **NFR-2.5** No API keys or secrets exist in the product; Claude Code handles the model connection under the student's login. `.gitignore` excludes `node_modules/`, `*.db` and `.env`.
- **NFR-2.6** Page text, résumé text and job descriptions are data, not instructions; text on a page can never add a tool call the student did not ask for. (SECURITY-11 misuse case: prompt injection through a job posting)
- **NFR-2.7** Shared rules are interpreted as the five fixed step types only; `eval` and `new Function` are prohibited. Downloaded rules are verified by schema and SHA-256 checksum before use. (SECURITY-13)
- **NFR-2.8** Dependencies are pinned with a committed `package-lock.json`, installed with `npm ci`, and checked with `npm audit` in CI; CI actions use pinned versions. (SECURITY-10)
- **NFR-2.9** Every database, file and network call has explicit error handling; on error the operation stops and the field is flagged (fail closed), and the MCP server has a top-level error handler that returns a generic message. (SECURITY-15)
- **NFR-2.10** Claude sees the page text and only the profile fields that page needs; the README states this plainly.

### NFR-3 Reliability
- **NFR-3.1** The plugin works offline with cached shared rules (FR-6.6).
- **NFR-3.2** A read-back mismatch always disables the responsible rule locally and flags the field (SC4).
- **NFR-3.3** Resiliency Baseline extension is not enabled for this local, single-user plugin. (Q22)

### NFR-4 Compatibility
- **NFR-4.1** Supported platforms: Windows and macOS, with Google Chrome and the Claude in Chrome extension, Node.js 22.13 or later, and Claude Code on a paid Claude plan. (Q16)
- **NFR-4.2** The MCP server ships as one bundled file (`server/dist/index.js`) using only the built-in `node:sqlite` module, so no native binaries are needed.
- **NFR-4.3** English-language desktop forms only.

### NFR-5 Testability and Maintainability
- **NFR-5.1** Every deterministic decision (field matching, planning, rule schema, personal-data scan, answer scope, word limits) has Vitest unit tests. (Q17)
- **NFR-5.2** Line coverage on `server/src` is at least 70%, enforced in CI. (Q17)
- **NFR-5.3** Property-based tests with fast-check cover round-trips (record and rule serialization, reference substitution and resolution) and invariants (the planner returns one action per field, sanitized rules contain no personal literal, drafts stay within word limits). PBT runs in Partial mode: PBT-02, PBT-03, PBT-07, PBT-08 and PBT-09 are blocking. (Q23)
- **NFR-5.4** Playwright replay tests run every shared rule against the three fixture forms for all three applicants in CI.
- **NFR-5.5** `claude plugin validate --strict` passes on the plugin manifest.

### NFR-6 Usability
- **NFR-6.1** Setup and filling are conversational inside Claude Code; every run ends with a review list the student can act on.
- **NFR-6.2** Error messages to the student are plain and generic, with a suggested next step.

---

## Constraints and Prohibited Choices (from tech-env.md)

- Claude Code plugin format with `.claude-plugin/plugin.json`, `skills/<name>/SKILL.md` and `.mcp.json`; no `bin/` folder and no name starting with `claude-`.
- TypeScript 5.x, ES modules, `@modelcontextprotocol/sdk` (stdio), Zod 3, `node:sqlite`, esbuild, npm.
- Prohibited: native database modules, Puppeteer or Selenium inside the plugin, OpenAI or Anthropic SDKs inside the plugin, `axios` or `node-fetch`, ATS or LinkedIn scraping libraries, `eval` or `new Function` on rule content.

## Out of Scope (MVP)

Automatic submission or signing; logging in, account creation or CAPTCHAs; email tracking, job ranking and interview preparation; job board scraping; guaranteed support for every real employer site; hosted backend or cloud sync; merging shared rules without review; non-English forms and mobile browsers; per-job cover letter writing; multiple profiles or résumé versions (Q10).

## Success Criteria Traceability (December 16 demo)

| Success criterion (vision.md) | Requirements |
| --- | --- |
| SC1: 3 applicants x 3 forms, at least 57 of 60+ field decisions correct, all wrong or unknown fields flagged | FR-2.2 to FR-2.8, FR-9.1, FR-9.2, NFR-1.1 |
| SC2: Applicant B needs fewer corrections than A on controls A taught | FR-5.1 to FR-5.4, FR-6.1 to FR-6.5 |
| SC3: No personal values in any shared rule | FR-5.3, FR-6.2, FR-6.3, NFR-2.7, NFR-5.3 |
| SC4: Changed fixture control is caught by read-back, rule disabled locally, field flagged | FR-2.4, FR-6.7, NFR-3.2 |
| SC5: Draft stays within word limit and is not typed before approval | FR-4.3, FR-4.4 |

## Open Items Carried Forward

- The Chrome plus MCP compatibility spike runs before Construction; the spike owner reports and the team decides on the fallback (Q20). Application Design must keep the fill-sheet fallback as an alternate path.

## Extension Compliance Summary (Requirements Analysis)

| Rule | Status | Note |
| --- | --- | --- |
| SECURITY-01 | Compliant (adapted) | NFR-2.2: local store protected by OS disk encryption and user-only permissions; HTTPS for rule sync |
| SECURITY-02 | N/A | No network intermediaries (no load balancer, gateway or CDN) |
| SECURITY-03 | Compliant (adapted) | NFR-2.4: structured local logs without PII; no centralized service exists for a local plugin |
| SECURITY-04 | N/A | No HTML-serving endpoints (fixture forms are test assets only) |
| SECURITY-05 | Compliant | NFR-2.3 |
| SECURITY-06 | N/A | No cloud IAM policies |
| SECURITY-07 | N/A | No network configuration |
| SECURITY-08 | N/A at this stage | Single local user; tools act only for the student who runs the session. Revisit in Application Design |
| SECURITY-09 | Compliant | No default credentials; generic error messages (NFR-6.2) |
| SECURITY-10 | Compliant | NFR-2.8 |
| SECURITY-11 | Compliant | Prompt-injection and submit-button misuse cases (NFR-2.6, FR-2.7) |
| SECURITY-12 | N/A | The product has no authentication of its own |
| SECURITY-13 | Compliant | NFR-2.7 checksums and safe interpretation |
| SECURITY-14 | N/A | No deployed service to monitor |
| SECURITY-15 | Compliant | NFR-2.9 |
| PBT-09 | Compliant | fast-check with Vitest selected (NFR-5.3) |
| PBT-02, 03, 07, 08 | Planned | Enforced in Functional Design and Code Generation |
