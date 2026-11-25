# Units of Work

**Version 2** (after the change request): six units, one foundation unit plus one unit per README work stream (unit plan Question 8: A). Tracking and ranking form U5; the shared fix library is removed; fixtures and demo move to U6.

The product is one deployable Claude Code plugin with one bundled MCP server, plus repository assets (CI, fixtures). Units are **modules for development**, not separately deployed services. Decisions come from `aidlc-docs/inception/plans/unit-of-work-plan.md` (Q1-Q8).

| Unit | Name | Components | Primary stories | README work stream |
| --- | --- | --- | --- | --- |
| U1 | Foundation and Contracts | C4 (skeleton), C14, C15, C12 (schema and fingerprints), C17 (CI skeleton), plugin packaging | none directly; enables all | built jointly by the team |
| U2 | Profile, Preferences and Answers | C1, C6, C7, C8, C9, C21 | 11 | Profile, preferences and answer store |
| U3 | Fill Planner and Skills | C2, C5, C10 | 9 | Fill planner and skills |
| U4 | Rule Learning | C3, C11, C12 (references, sanitizer, scan), C17 (replay job) | 4 | Rule learning |
| U5 | Tracking and Ranking | C19, C20, C22, C23 | 11 | Application tracking and job ranking |
| U6 | Fixtures, Tests and Demo | C18 and the scenario test plan | 4 | Fixtures and tests |

---

## U1 - Foundation and Contracts

**Purpose**: Make every other unit buildable in parallel and retire the riskiest assumption first. Built jointly by the team in the first Construction week.

**Responsibilities**
- Chrome plus MCP spike: a hello-world plugin with one MCP tool used in the same session as Claude in Chrome; the team decides on the fallback from the result (requirements Q20).
- Plugin scaffold: `.claude-plugin/plugin.json`, `.mcp.json`, five skill folders, esbuild bundle to `server/dist/index.js`, pinned `package.json` and `package-lock.json`.
- Shared contracts in `server/src/types.ts` and `server/src/core/schema.ts`: fill types (`Field`, `PlannedAction`, `ReadBack`, `Rule`, `RunState`), tracking types (`Status`, `Application`, `StatusProposal`), ranking types (`Questionnaire`, `PostingFacts`, `Judgement`, `FitResult`), `Result` and `ErrorCode`, with Zod schemas.
- `server/src/core/fingerprint.ts`: control and page fingerprints.
- C4 registration pattern and global error handler; all 29 tools registered with input schemas and `NOT_IMPLEMENTED` stubs.
- C14 repository: database open, migrations for every table, parameterized helpers, `deleteAll`.
- C15 logger with an allowlist.
- CI skeleton: `npm ci`, build, tests with coverage, `npm audit`, `claude plugin validate --strict`.

## U2 - Profile, Preferences and Answers

**Purpose**: Everything the student tells the plugin about themselves, stored safely and returned correctly.

**Responsibilities**
- C1 Set-Up-Profile Skill: résumé intake, extraction, section confirmation, cover letter, questionnaire, sensitive answers, delete-my-data flow.
- C6 Profile Service, C7 Document Registry, C8 Answer Service, C9 Sensitive Answer Guard, C21 Questionnaire Service.
- Tools: `register_document`, `get_document`, `save_profile_records`, `confirm_profile_section`, `get_profile`, `set_sensitive_answer`, `save_questionnaire`, `get_questionnaire`, `find_answers`, `check_answer`, `save_answer`, `delete_my_data`.

## U3 - Fill Planner and Skills

**Purpose**: Turn an open application page into a verified, reviewable fill that lands in the tracker.

**Responsibilities**
- C10 Fill Planner, C5 Run Manager, C2 Fill-Application Skill (including fill-sheet fallback mode and the `log_application` call after a fill).
- Tools: `plan_fill`, `record_fill_result`, `undo_page`, `close_run`.
- End-to-end integration of U2, U4 and U5 into the fill flow.

## U4 - Rule Learning

**Purpose**: Learn a fix once and reuse it safely.

**Responsibilities**
- C12 (rest of the Rule Core): reference resolution, sanitizer, personal-data scan.
- C11 Rule Service: personal rules, disabling on read-back failure.
- C3 Teach-a-Fix Skill.
- C17 replay job: Playwright step executor using C12; replay of example taught rules on the fixture forms for applicants A, B, C.
- Tools: `propose_rule`, `list_rules`.

## U5 - Tracking and Ranking

**Purpose**: One tracker of every application, and an explainable fit score for every saved posting.

**Responsibilities**
- C22 Tracker Service: link deduplication, status transitions, student-only Submitted, auto-apply versus pending, duplicate messages, undo, No response at read time, list with filters and timelines, CSV export.
- C23 Fit Scorer: saved jobs, hard constraints, unknowns, evidence check, cap at 35, weights, rankings, Ranked status through C22, comparison.
- C19 Track-Applications Skill (message classification prompts, pending review, table rendering, Gmail as a stretch goal) and C20 Rank-This-Job Skill (fact extraction, judgements with quotes, explanations).
- Tools: `log_application`, `confirm_submission`, `update_status`, `list_pending_updates`, `resolve_pending_update`, `undo_last_change`, `list_applications`, `export_tracker_csv`, `save_job`, `score_job`, `compare_jobs`.

## U6 - Fixtures, Tests and Demo

**Purpose**: Make every success criterion measurable and the demo repeatable.

**Responsibilities**
- Three fixture forms with the seven hard controls and a changed-control variant; three synthetic applicants; an answer key of at least 60 field decisions.
- At least 20 labeled synthetic employer messages; at least 10 synthetic postings with two questionnaires, expected deal-breakers and team-agreed top-3.
- C18 Demo Scorer and the scripted scenario run that reports SC1-SC7; a local static server for fixtures.

---

## Code Organization Strategy (greenfield)

One npm package, following `tech-env.md`:

```text
plugins/job-application-assistant/
  .claude-plugin/plugin.json          U1
  .mcp.json                           U1
  skills/set-up-profile/SKILL.md      U2
  skills/fill-application/SKILL.md    U3
  skills/teach-a-fix/SKILL.md         U4
  skills/track-applications/SKILL.md  U5
  skills/rank-this-job/SKILL.md       U5
  server/
    package.json, package-lock.json   U1
    src/index.ts                      U1 (registration), units add handlers
    src/types.ts                      U1
    src/core/schema.ts                U1
    src/core/fingerprint.ts           U1
    src/core/refs.ts                  U4
    src/core/sanitize.ts              U4
    src/db.ts                         U1
    src/log.ts                        U1
    src/profile.ts                    U2
    src/documents.ts                  U2
    src/answers.ts                    U2
    src/sensitive.ts                  U2
    src/questionnaire.ts              U2
    src/plan.ts                       U3
    src/run.ts                        U3
    src/rules.ts                      U4
    src/tracker.ts                    U5
    src/csv.ts                        U5
    src/score.ts                      U5
    test/*.test.ts                    each unit for its modules
    dist/index.js                     U1 build, committed
tests/replay/                         U4
fixtures/forms/                       U6
fixtures/applicants/                  U6
fixtures/messages/                    U6
fixtures/postings/                    U6
fixtures/answer-key.json              U6
tools/score-demo.ts                   U6
.github/workflows/ci.yml              U1 (skeleton), U4 (replay job)
```

No `bin/` folder and no name starting with `claude-`. Application code never goes in `aidlc-docs/`.

## Ownership (Q5, Q8)

U1 is built jointly in the first Construction week. Owners and reviewers for U2-U6 are assigned at the first Construction meeting and recorded in `aidlc-state.md`. Each of the five teammates owns one unit and reviews a different one.

| Unit | Owner | Reviewer |
| --- | --- | --- |
| U1 | whole team | whole team |
| U2 | to be assigned | to be assigned |
| U3 | to be assigned | to be assigned |
| U4 | to be assigned | to be assigned |
| U5 | to be assigned | to be assigned |
| U6 | to be assigned | to be assigned |

## Definition of Done (Q7)

1. Every acceptance criterion of the unit's mapped stories passes (`unit-of-work-story-map.md`).
2. Unit and property tests pass with at least 70% line coverage for its modules; PBT-02, 03, 07, 08 and 09 met where they apply.
3. Its Security Baseline compliance summary has no blocking findings.
4. Its pull request is approved by a teammate other than the owner.

## Per-Unit Construction Stages

Each unit runs Functional Design, NFR Requirements, NFR Design and Code Generation (Infrastructure Design skipped, per `execution-plan.md`). U1's Functional Design covers contracts and schema only.
