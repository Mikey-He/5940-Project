# Application Design - Job Application Assistant

This document consolidates `components.md`, `component-methods.md`, `services.md` and `component-dependency.md`. Detailed business rules are deferred to Functional Design (per unit, Construction).

## Architecture Overview

```text
+--------------------------------------------------------------+
| Claude Code session (student's own Claude plan)              |
|                                                              |
| Skills: C1 set-up-profile, C2 fill-application, C3 teach     |
|    |                            |                            |
|    | MCP tools (stdio)          | Claude in Chrome tools     |
|    v                            v                            |
|  +------------------------+  +-------------------------+     |
|  | Local MCP server       |  | Employer page in Chrome |     |
|  | C4 Tool Layer          |  +-------------------------+     |
|  | C5 Run, C6 Profile     |                                  |
|  | C7 Docs, C8 Answers    |                                  |
|  | C9 Sensitive Guard     |                                  |
|  | C10 Planner, C11 Rules |                                  |
|  | C12 Rule Core (shared) |                                  |
|  | C13 Sync, C15 Logger   |                                  |
|  | C14 Repository         |                                  |
|  +-----------+------------+                                  |
+--------------|-----------------------------------------------+
               |              ^                                 
               v              | HTTPS (read-only)               
  SQLite in plugin data   GitHub: C16 rules/approved            
                          C17 CI replay, C18 fixtures           
```

Text alternative: inside a Claude Code session, three skills drive the work. They call the local MCP server over stdio and operate the employer page through Claude in Chrome. The server's tool layer calls domain components, which store data through one repository module in a local SQLite database. Approved shared rules come from the public GitHub folder over HTTPS; CI replays rules against fixture forms before they are merged.

## Key Design Decisions

| # | Decision | Reason |
| --- | --- | --- |
| Q1 | Thin tool handlers -> domain modules -> one repository module | Validation at the edge; all SQL parameterized in one place |
| Q2 | Skills orchestrate; server enforces a run state machine (`planned -> filled -> verified -> closed`) | Safety holds even if Claude skips a step (fail closed) |
| Q3 | Synonym-table matching only; ambiguous labels go to Claude with a visible mark | "Flagged, never guessed" |
| Q4 | Page fingerprint for history; control fingerprint for rule matching | A fix taught on one form works on the same kind of control elsewhere |
| Q5 | Undo originals held in memory for the active page only | No values in history |
| Q6 | Code classifies sensitive fields; Claude can only add restriction | Sensitive answers never guessed |
| Q7 | Structured results with stable error codes; global handler | Skills react to `FILE_CHANGED`, `RUN_STATE_INVALID`, etc.; generic messages |
| Q8 | One shared Rule Core used by server and CI | CI tests exactly what students run |
| Q9 | Fill-sheet fallback is a mode of the fill skill | Same planning and safety rules |
| Q10 | JSON-lines logs to file and stderr | stdout carries MCP |
| Q11 | Unit and property tests with in-memory SQLite; in-process MCP integration tests | Fast, isolated, contract-level coverage |

## Components (summary)

| ID | Component | Responsibility | Stories |
| --- | --- | --- | --- |
| C1 | Set-Up-Profile Skill | Résumé to confirmed profile; documents; sensitive answers; delete data | US-1.x, US-3.1, US-8.1 |
| C2 | Fill-Application Skill | Plan, fill, verify, review, undo, next page; fill-sheet mode | US-2.x, US-3.2, US-4.x, US-7.x, US-9.1, 9.3, 9.4 |
| C3 | Teach-a-Fix Skill | Demonstration to rule; export; failure issue | US-5.x, US-6.1, US-9.2 |
| C4 | Tool Layer | 19 MCP tools, Zod validation, error mapping | All |
| C5 | Run Manager | State machine, in-memory undo, history without values | US-2.1, 2.4, 7.x |
| C6 | Profile Service | Records, dedupe, section confirmation | US-1.2, 1.3 |
| C7 | Document Registry | Path, size, SHA-256; change detection | US-1.1, 1.4, 2.3, 9.1 |
| C8 | Answer Service | Scoped answers, age, word counts | US-4.x |
| C9 | Sensitive Answer Guard | Classification and exact-match answers | US-3.x, 2.2 |
| C10 | Fill Planner | One action per field | US-2.x, 5.2 |
| C11 | Rule Service | Personal rules, share eligibility, disabling | US-5.1, 6.1, 9.2 |
| C12 | Rule Core (shared) | Schema, fingerprints, references, sanitizer, scan | US-5.1, 6.x; SC3 |
| C13 | Rule Sync | HTTPS index, checksum, cache, offline | US-6.3, 6.4, 9.3 |
| C14 | Repository | Only `node:sqlite` access; migrations; delete-all | All data stories |
| C15 | Logger | Allowlisted JSON-lines logs | US-7.1 |
| C16 | Shared Rule Library | `rules/approved/` with checksummed index | US-6.2-6.4 |
| C17 | Replay Harness and CI | Playwright executor, checks, pinned CI | US-6.2, 10.3 |
| C18 | Fixtures and Demo Scorer | Forms, applicants, answer key, SC report | US-10.1, 10.2, 10.4 |

## MCP Tools
`register_document`, `get_document`, `save_profile_records`, `confirm_profile_section`, `get_profile`, `set_sensitive_answer`, `plan_fill`, `find_answers`, `check_answer`, `save_answer`, `record_fill_result`, `undo_page`, `close_run`, `propose_rule`, `list_share_candidates`, `export_rule_candidate`, `draft_rule_issue`, `sync_rules`, `delete_my_data`. Signatures in `component-methods.md`.

## Services
S1 Profile Setup, S2 Fill (per page, state machine), S3 Fill-Sheet Fallback, S4 Teach and Share, S5 Rule Review and Publication, S6 Rule Sync, S7 Data Deletion, S8 Demo Scoring. Steps in `services.md`.

## Dependencies
Skills -> C4 only; C4 -> domain components; domain components -> C14 only for data; C10 and C12 are pure and shared with tests; no cycles. Matrix and data-flow diagrams in `component-dependency.md`.

## Completeness Check
- Every story US-1.1 to US-10.4 maps to at least one component and one service (see the tables above and in `components.md`).
- Consistent with `tech-env.md`: plugin layout, `node:sqlite`, Zod, esbuild single file, global `fetch`, prohibited libraries respected. The tool list extends the 11 tools in `tech-env.md` with eight tools required by stories.
- The Chrome plus MCP risk has a designed alternate path (S3).

## Extension Compliance Summary (Application Design)

| Rule | Status | Note |
| --- | --- | --- |
| SECURITY-01 | Compliant (adapted) | Local SQLite in the user-only plugin data folder on an OS-encrypted disk; HTTPS for sync |
| SECURITY-03 | Compliant (adapted) | C15 structured, allowlisted logs to local file and stderr |
| SECURITY-05 | Compliant | C4 Zod schemas with bounds on every tool; C14 parameterized queries |
| SECURITY-08 | Compliant | Tools act only on the local student's own data; no cross-user resources; destructive tools require explicit `confirm` / `consent` |
| SECURITY-09 | Compliant | No default credentials; generic error messages |
| SECURITY-10 | Compliant | Lock file, `npm ci`, `npm audit` and pinned CI in C17 |
| SECURITY-11 | Compliant | Security-critical logic isolated (C9, C12, C14); defense in depth (validation + state machine + read-back); misuse cases: prompt injection, submit click, personal data in rules |
| SECURITY-13 | Compliant | Checksum and schema verification in C13; no `eval`; CI changes reviewed |
| SECURITY-15 | Compliant | Structured errors, global handler, fail-closed state machine, transactions with rollback |
| SECURITY-02, 04, 06, 07, 12, 14 | N/A | No intermediaries, HTML endpoints, cloud IAM, network config, authentication or deployed monitoring |
| PBT-09 | Compliant | fast-check with Vitest (Q11) |
| PBT-02, 03, 07, 08 | Planned | Pure components C10, C12, C8 are designed for properties; enforced in Functional Design and Code Generation |
