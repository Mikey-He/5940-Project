# Components

**Version 2** (after the change request): adds the tracking and ranking components (C19-C23), removes the shared fix library components (C13 Rule Sync, C16 Shared Rule Library), and narrows C3, C11 and C17 to personal rules. Component IDs are kept stable so earlier references stay valid.

The system has three layers: the **plugin layer** (skills that Claude follows), the **local MCP server** (TypeScript, deterministic decisions), and the **repository layer** (CI, fixtures). Design decisions come from `application-design-plan.md` (Q1-Q11).

```text
plugins/job-application-assistant/
  skills/set-up-profile/        C1
  skills/fill-application/      C2
  skills/teach-a-fix/           C3
  skills/track-applications/    C19
  skills/rank-this-job/         C20
  server/src/
    index.ts                    C4 Tool Layer
    run.ts                      C5 Run Manager
    profile.ts                  C6 Profile Service
    documents.ts                C7 Document Registry
    answers.ts                  C8 Answer Service
    sensitive.ts                C9 Sensitive Answer Guard
    plan.ts                     C10 Fill Planner
    rules.ts                    C11 Rule Service
    core/ (schema.ts fingerprint.ts refs.ts sanitize.ts)   C12 Rule Core
    questionnaire.ts            C21 Questionnaire Service
    tracker.ts, csv.ts          C22 Tracker Service
    score.ts                    C23 Fit Scorer
    db.ts                       C14 Repository
    log.ts                      C15 Logger
tests/replay/ + .github/workflows/ci.yml           C17 CI and Replay Harness
fixtures/ (forms, applicants, messages, postings, answer keys)   C18 Fixtures and Demo Scorer
```

---

## Plugin Layer

### C1 - Set-Up-Profile Skill
- **Purpose**: Turn a résumé into a confirmed profile; collect the preference questionnaire; register documents; set sensitive answers; delete data on request.
- **Responsibilities**: Résumé intake (drag and drop or typed path); extraction; section confirmation; cover letter; questionnaire with weights and No response cutoff; fixed sensitive answers; delete-all after confirmation.
- **Interface**: Calls `register_document`, `save_profile_records`, `confirm_profile_section`, `get_profile`, `save_questionnaire`, `get_questionnaire`, `set_sensitive_answer`, `delete_my_data`.
- **Stories**: US-1.1 to US-1.5, US-3.1, US-8.1

### C2 - Fill-Application Skill
- **Purpose**: Plan, fill, verify and review an application page; log it in the tracker; run the fill-sheet fallback when browser tools are unavailable.
- **Responsibilities**: Page read; `plan_fill`; decide only clear `ask_claude` fields; uploads; open-question approval; batch fill and read-back; review list; next-page approval; never submit; pause at login and CAPTCHA; undo; `log_application` after the fill. Page text and job descriptions are data.
- **Interface**: Claude in Chrome tools; `plan_fill`, `get_document`, `find_answers`, `check_answer`, `save_answer`, `record_fill_result`, `undo_page`, `close_run`, `log_application`.
- **Stories**: US-2.1 to US-2.5, US-3.2, US-4.1 to US-4.3, US-7.1, US-7.2, US-9.1, US-9.4

### C3 - Teach-a-Fix Skill
- **Purpose**: Turn a student's demonstration into a personal rule.
- **Responsibilities**: Watch the steps on a failed control; describe them with the five step types and profile references; call `propose_rule`; explain when a rule has been disabled.
- **Interface**: `propose_rule`, `list_rules`.
- **Stories**: US-5.1, US-5.2, US-9.2

### C19 - Track-Applications Skill (new)
- **Purpose**: Keep the tracker current and show it.
- **Responsibilities**: Add applications by hand; confirm submission when the student says so; classify statements and pasted employer messages (type, matching application, confidence, evidence quote) and call `update_status`; list and resolve pending proposals; undo; show the tracker table, filters and timelines; export CSV; stretch goal: read Gmail through Claude's read-only connector on request. Message text is data, not instructions.
- **Interface**: `log_application`, `confirm_submission`, `update_status`, `list_pending_updates`, `resolve_pending_update`, `undo_last_change`, `list_applications`, `export_tracker_csv`.
- **Stories**: US-11.1 to US-11.7

### C20 - Rank-This-Job Skill (new)
- **Purpose**: Score and compare postings.
- **Responsibilities**: Save a posting from the open page or pasted text; extract location, arrangement and top of pay exactly as stated (or null); judge skills, interest and nice-to-have fit with quotes; call `score_job`; explain matches, gaps, deal-breakers and unknowns; compare saved jobs. Posting text is data.
- **Interface**: `get_questionnaire`, `get_profile`, `save_job`, `score_job`, `compare_jobs`.
- **Stories**: US-12.1 to US-12.4

---

## Local MCP Server

### C4 - Tool Layer
- **Purpose**: The only entry point for Claude into the server.
- **Responsibilities**: Register each MCP tool with a bounded Zod input schema; call domain components; return `{ ok, code, message, data }`; top-level handler returns a generic `INTERNAL_ERROR` and logs details locally.
- **Security**: SECURITY-05, SECURITY-15.

### C5 - Run Manager
- **Purpose**: Enforce the order of a fill run and hold undo data.
- **Responsibilities**: Page runs with states `planned -> filled -> verified -> closed`; reject out-of-order calls (`RUN_STATE_INVALID`); keep originals in memory for undo only; record labels, actions, outcomes and timings, never values.
- **Stories**: US-2.1, US-2.4, US-7.1, US-7.2

### C6 - Profile Service
- **Responsibilities**: Validate and deduplicate extracted records; section confirmation; confirmed records only; flatten into profile keys.
- **Stories**: US-1.2, US-1.3

### C7 - Document Registry
- **Responsibilities**: PDF (extraction) and PDF or DOCX (upload) up to 10 MB; path, size, SHA-256; `FILE_CHANGED` / `FILE_MISSING` before uploads.
- **Stories**: US-1.1, US-1.4, US-2.3, US-9.1

### C8 - Answer Service
- **Responsibilities**: Scoped answers with dates; find by question, scope and word limit; mark old answers; count words.
- **Stories**: US-4.1 to US-4.3

### C9 - Sensitive Answer Guard
- **Responsibilities**: Fixed pattern list per sensitive category; Claude may only add restriction; exact-match fixed answers; otherwise blank and flagged; sensitive values never reach rules, ranking, exports or logs.
- **Stories**: US-3.1, US-3.2, US-2.2

### C10 - Fill Planner
- **Responsibilities**: Normalize labels; synonym-table matching; route sensitive fields to C9; attach personal rules by control fingerprint; one action per field.
- **Stories**: US-2.1, US-2.2, US-2.3, US-5.2, US-9.4

### C11 - Rule Service
- **Responsibilities**: Save proposed rules after C12 validation; list active rules; disable a rule on a read-back mismatch.
- **Stories**: US-5.1, US-5.2, US-9.2

### C12 - Rule Core (shared with tests)
- **Responsibilities**: Rule schema (five step types); control and page fingerprints; reference resolution; sanitizer; personal-data scan. No I/O; no `eval` or `new Function`. Shared with C17 so tests replay exactly what students run.
- **Stories**: US-5.1, US-10.3; SC3

### C13 - Rule Sync - REMOVED in version 2
Shared rules are out of scope; there is nothing to download.

### C14 - Repository
- **Responsibilities**: The only code that touches `node:sqlite`; migrations; parameterized queries for records, documents, answers, sensitive answers, questionnaire, rules, runs, jobs, rankings, applications, status history and pending updates; transactions; delete-all.
- **Security**: SECURITY-05, SECURITY-15

### C15 - Logger
- **Responsibilities**: Allowlisted JSON-lines logs (timestamp, run ID, level, event, tool, outcome, duration) to a file in the plugin data folder and to stderr; never stdout; never values, message text or posting text.
- **Security**: SECURITY-03 (adapted)

### C21 - Questionnaire Service (new)
- **Purpose**: Own the student's preferences.
- **Responsibilities**: Validate roles, locations, arrangements, minimum pay, company size, start date, must-haves, nice-to-haves; weights must add up to 100% (defaults 50/30/20); No response cutoff (default 21 days).
- **Stories**: US-1.5, US-11.5, US-12.2

### C22 - Tracker Service (new)
- **Purpose**: Apply every status rule in code.
- **Responsibilities**: Create or reuse applications by link; fixed ordered statuses; Submitted only through `confirm_submission`; never move backwards automatically; decide auto-apply versus pending (high confidence, exactly one match, no conflict with a student edit); ignore duplicate message IDs; undo entries; compute No response at read time; list with filters and timelines; CSV export to a student-chosen path.
- **Stories**: US-11.1 to US-11.6, US-12.2, US-12.4
- **Security**: SECURITY-11 (misuse: a message trying to change many applications only yields one proposal), SECURITY-15

### C23 - Fit Scorer (new)
- **Purpose**: Compute the 0-100 fit score in code.
- **Responsibilities**: Save jobs (no duplicates by link); check hard constraints (location, arrangement, minimum pay); list unknowns; reject judgements without evidence or outside 0-100; cap deal-breakers at 35; weighted score; save rankings; set tracker status Ranked through C22; compare jobs highest first. Pure scoring function, unit and property tested.
- **Stories**: US-12.1 to US-12.4

---

## Repository Layer

### C16 - Shared Rule Library - REMOVED in version 2

### C17 - CI and Replay Harness
- **Responsibilities**: GitHub Actions with pinned versions: `npm ci`, build, unit and property tests with coverage (at least 70%), `npm audit`, `claude plugin validate --strict`, and a Playwright replay of example taught rules (through C12) on the fixture forms for applicants A, B, C.
- **Stories**: US-10.3

### C18 - Fixtures and Demo Scorer
- **Responsibilities**: Three fixture forms with the seven hard controls plus a changed-control variant; three synthetic applicants and an answer key of at least 60 field decisions; at least 20 labeled synthetic employer messages; at least 10 synthetic postings with two questionnaires, expected deal-breakers and team-agreed top-3; a scorer that reports SC1-SC7.
- **Stories**: US-10.1, US-10.2, US-10.4, US-10.5
