# Units of Work

The product is one deployable Claude Code plugin with one bundled MCP server, plus repository assets (shared rules, CI, fixtures). The five units below are **modules for development**, not separately deployed services. Decisions come from `aidlc-docs/inception/plans/unit-of-work-plan.md` (Q1-Q7).

| Unit | Name | Components | Primary stories | README work stream |
| --- | --- | --- | --- | --- |
| U1 | Foundation and Contracts | C4 (skeleton), C14, C15, C12 (schema and fingerprints), plugin packaging | none directly; enables all | shared by the team |
| U2 | Profile and Answer Store | C1, C6, C7, C8, C9 | 10 | Profile and answer store |
| U3 | Fill Planner and Skills | C2, C5, C10 | 9 | Fill planner and skills |
| U4 | Rule Learning and Validation | C3, C11, C12 (sanitizer, scan, references), C13, C16, C17 | 9 | Rule learning and validation |
| U5 | Fixtures, Tests and Demo | C18 and the scenario test plan | 3 | Fixtures and tests |

---

## U1 - Foundation and Contracts

**Purpose**: Make every other unit buildable in parallel and retire the riskiest assumption first.

**Responsibilities**
- Run the Chrome plus MCP spike: a hello-world plugin with one MCP tool, used in the same Claude Code session as Claude in Chrome. The owner reports the result and the team decides on the fallback (requirements Q20).
- Plugin scaffold: `.claude-plugin/plugin.json`, `.mcp.json`, three empty skill folders, esbuild bundle to `server/dist/index.js`, `package.json` with pinned dependencies and `package-lock.json`.
- Shared contracts in `server/src/types.ts` and `server/src/core/schema.ts`: `Field`, `PlannedAction`, `ReadBack`, `Result`, `ErrorCode`, `Rule`, `RuleStep`, `RunState`, with Zod schemas.
- `server/src/core/fingerprint.ts`: control and page fingerprint functions.
- C4 tool registration pattern and global error handler; all 19 tools registered with input schemas and `NOT_IMPLEMENTED` stubs.
- C14 repository: database open, migrations for all tables, parameterized query helpers, `deleteAll`.
- C15 logger with an allowlist.
- CI skeleton: `npm ci`, `npm run build`, `npm test` with coverage, `npm audit`, `claude plugin validate --strict`.

**Deliverables**: spike report; buildable plugin that loads with `claude --plugin-dir`; contracts merged before other units start integration.

## U2 - Profile and Answer Store

**Purpose**: Everything the student tells the plugin, stored safely and returned correctly.

**Responsibilities**
- C1 Set-Up-Profile Skill: résumé intake (drag and drop or typed path), extraction prompts, section confirmation, cover letter, sensitive answers, delete-my-data flow.
- C6 Profile Service: validation, deduplication, section confirmation, flattening to profile keys.
- C7 Document Registry: type and size checks, SHA-256, change detection.
- C8 Answer Service: scoped answers, season age, word counting.
- C9 Sensitive Answer Guard: category patterns, exact-match answers, restriction-only overrides.
- Tool implementations: `register_document`, `get_document`, `save_profile_records`, `confirm_profile_section`, `get_profile`, `set_sensitive_answer`, `find_answers`, `check_answer`, `save_answer`, `delete_my_data`.

## U3 - Fill Planner and Skills

**Purpose**: Turn an open application page into a verified, reviewable fill.

**Responsibilities**
- C10 Fill Planner: label normalization, synonym table, one action per field, routing to sensitive guard and rules.
- C5 Run Manager: state machine, in-memory undo originals, history without values.
- C2 Fill-Application Skill: page read, `ask_claude` policy, uploads, open-question approval, batch fill and read-back, review list, next-page approval, submit and login boundaries, undo, fill-sheet fallback mode.
- Tool implementations: `plan_fill`, `record_fill_result`, `undo_page`, `close_run`.
- Integration of U2 and U4 components into the end-to-end fill flow.

## U4 - Rule Learning and Validation

**Purpose**: Learn fixes once, share them safely, and keep broken ones out.

**Responsibilities**
- C12 (rest of the Rule Core): reference resolution, sanitizer, personal-data scan.
- C11 Rule Service: personal rules, clean read-back counting, disabling, share eligibility, issue drafts.
- C3 Teach-a-Fix Skill: demonstration capture, proposal, export consent.
- C13 Rule Sync: HTTPS index, checksum and schema verification, cache, offline notice.
- C16 Shared Rule Library: `rules/approved/index.json` format with SHA-256 per rule.
- C17 Replay Harness and CI: Playwright step executor using C12, replay for applicants A, B, C on the three forms, branch protection with one non-author approval.
- Tool implementations: `propose_rule`, `list_share_candidates`, `export_rule_candidate`, `draft_rule_issue`, `sync_rules`.

## U5 - Fixtures, Tests and Demo

**Purpose**: Make the success criteria measurable and the demo repeatable.

**Responsibilities**
- Three static fixture forms in `fixtures/forms/` with the seven hard controls, plus a changed-control variant for SC4.
- Three synthetic applicants in `fixtures/applicants/` with synthetic résumés (PDF) and expected values.
- `fixtures/answer-key.json` with at least 60 field decisions.
- C18 Demo Scorer and the scripted scenario run that reports SC1-SC5.
- A local static server script for fixtures used by Playwright and manual demo runs.

---

## Code Organization Strategy (greenfield)

One npm package, following `tech-env.md` (Q6):

```text
plugins/job-application-assistant/
  .claude-plugin/plugin.json          U1
  .mcp.json                           U1
  skills/set-up-profile/SKILL.md      U2
  skills/fill-application/SKILL.md    U3
  skills/teach-a-fix/SKILL.md         U4
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
    src/plan.ts                       U3
    src/run.ts                        U3
    src/rules.ts                      U4
    src/sync.ts                       U4
    test/*.test.ts                    each unit for its modules
    dist/index.js                     U1 build, committed
rules/approved/index.json             U4
tests/replay/                         U4
fixtures/forms/                       U5
fixtures/applicants/                  U5
fixtures/answer-key.json              U5
tools/score-demo.ts                   U5
.github/workflows/ci.yml              U1 (skeleton), U4 (replay job)
```

No `bin/` folder and no name starting with `claude-`. Application code never goes in `aidlc-docs/`.

## Ownership (Q5)

Owners and reviewers are assigned at the first Construction meeting and recorded in `aidlc-state.md`. Each of the five teammates owns one unit and reviews a different one.

| Unit | Owner | Reviewer |
| --- | --- | --- |
| U1 | to be assigned | to be assigned |
| U2 | to be assigned | to be assigned |
| U3 | to be assigned | to be assigned |
| U4 | to be assigned | to be assigned |
| U5 | to be assigned | to be assigned |

## Definition of Done (Q7)

A unit is complete when:
1. Every acceptance criterion of its mapped stories passes (see `unit-of-work-story-map.md`).
2. Unit and property tests pass, with at least 70% line coverage for its modules; PBT-02, 03, 07, 08 and 09 are met where they apply.
3. Its Security Baseline compliance summary has no blocking findings.
4. Its pull request is approved by a teammate other than the owner.

## Per-Unit Construction Stages

Each unit runs Functional Design, NFR Requirements, NFR Design and Code Generation (Infrastructure Design is skipped, per `execution-plan.md`). U1's Functional Design is short (contracts and schema only).
