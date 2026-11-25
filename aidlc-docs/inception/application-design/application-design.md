# Application Design - Job Application Assistant

**Version 2** (after the change request that added application tracking and job ranking and removed the shared fix library). This document consolidates `components.md`, `component-methods.md`, `services.md` and `component-dependency.md`. Detailed business rules are deferred to Functional Design (per unit, Construction).

## Architecture Overview

```text
+----------------------------------------------------------------+
| Claude Code session (student's own Claude plan)                |
|                                                                |
| Skills: C1 profile, C2 fill, C3 teach, C19 track, C20 rank     |
|    |                          |                  |             |
|    | MCP tools (stdio)        | Claude in Chrome | Gmail (opt.)|
|    v                          v                  v             |
|  +------------------------+  +----------------+  +----------+  |
|  | Local MCP server       |  | Employer page  |  | Read-only|  |
|  | C4 Tool Layer          |  +----------------+  +----------+  |
|  | C5 Run, C6 Profile     |                                    |
|  | C7 Docs, C8 Answers    |                                    |
|  | C9 Sensitive Guard     |                                    |
|  | C10 Planner, C11 Rules |                                    |
|  | C12 Rule Core          |                                    |
|  | C21 Questionnaire      |                                    |
|  | C22 Tracker            |                                    |
|  | C23 Fit Scorer         |                                    |
|  | C14 Repository, C15 Log|                                    |
|  +-----------+------------+                                    |
+--------------|-------------------------------------------------+
               v                                                  
  SQLite in the plugin data folder (no network, no ports)         
```

Text alternative: inside a Claude Code session, five skills drive the work: set up profile and preferences, fill an application, teach a fix, track applications, and rank a job. They call the local MCP server over stdio, operate the employer page through Claude in Chrome, and (stretch goal) read employer replies through Claude's read-only Gmail connector. The server's tool layer calls domain components, which store everything through one repository module in a local SQLite database. The server makes no network requests and opens no ports.

## Key Design Decisions

| # | Decision | Reason |
| --- | --- | --- |
| Q1 | Thin tool handlers -> domain modules -> one repository module | Validation at the edge; all SQL parameterized in one place |
| Q2 | Skills orchestrate; the server enforces rules (fill run state machine, status transitions, scoring constraints) | Safety holds even if Claude skips a step (fail closed) |
| Q3 | Synonym-table matching only; ambiguous labels go to Claude with a visible mark | "Flagged, never guessed" |
| Q4 | Page fingerprint for history; control fingerprint for rule matching | A fix taught on one form works on the same kind of control elsewhere |
| Q5 | Undo originals held in memory for the active page only | No values in history |
| Q6 | Code classifies sensitive fields; Claude can only add restriction | Sensitive answers never guessed |
| Q7 | Structured results with stable error codes; global handler | Skills react to `FILE_CHANGED`, `RUN_STATE_INVALID`, `EVIDENCE_MISSING`, and so on |
| Q8 | One Rule Core shared by the server and the CI replay | Tests replay exactly what students run |
| Q9 | Fill-sheet fallback is a mode of the fill skill | Same planning and safety rules |
| Q10 | JSON-lines logs to file and stderr | stdout carries MCP |
| Q11 | Unit and property tests with in-memory SQLite; in-process MCP integration tests | Fast, isolated, contract-level coverage |
| v2 | Tracking and ranking follow Q1, Q2, Q7, Q10, Q11: Claude classifies and judges with evidence; C22 and C23 decide in code | Same LLM-interprets, code-decides rule as filling |

## Components (summary)

| ID | Component | Responsibility | Stories |
| --- | --- | --- | --- |
| C1 | Set-Up-Profile Skill | Résumé to confirmed profile; questionnaire; documents; sensitive answers; delete data | US-1.x, US-3.1, US-8.1 |
| C2 | Fill-Application Skill | Plan, fill, verify, review, undo, next page, log application; fill-sheet mode | US-2.x, US-3.2, US-4.x, US-7.x, US-9.1, US-9.4 |
| C3 | Teach-a-Fix Skill | Demonstration to personal rule | US-5.x, US-9.2 |
| C19 | Track-Applications Skill | Manual entries, submission, status updates, pending, undo, table, CSV, Gmail (stretch) | US-11.x |
| C20 | Rank-This-Job Skill | Save postings, extract facts, judge fit with evidence, explain, compare | US-12.x |
| C4 | Tool Layer | 29 MCP tools, Zod validation, error mapping | All |
| C5 | Run Manager | Fill run state machine, in-memory undo, history without values | US-2.1, 2.4, 7.x |
| C6 | Profile Service | Records, dedupe, section confirmation | US-1.2, 1.3 |
| C7 | Document Registry | Path, size, SHA-256; change detection | US-1.1, 1.4, 2.3, 9.1 |
| C8 | Answer Service | Scoped answers, age, word counts | US-4.x |
| C9 | Sensitive Answer Guard | Classification and exact-match answers | US-3.x, 2.2 |
| C10 | Fill Planner | One action per field | US-2.x, 5.2 |
| C11 | Rule Service | Personal rules; disable on failure | US-5.x, 9.2 |
| C12 | Rule Core | Schema, fingerprints, references, sanitizer, scan | US-5.1, 10.3; SC3 |
| C14 | Repository | Only `node:sqlite` access; migrations; delete-all | All data stories |
| C15 | Logger | Allowlisted JSON-lines logs | US-7.1 |
| C21 | Questionnaire Service | Preferences, weights, No response cutoff | US-1.5, 11.5, 12.2 |
| C22 | Tracker Service | Statuses, transitions, auto-apply versus pending, undo, No response, CSV | US-11.x, 12.2, 12.4 |
| C23 | Fit Scorer | Jobs, hard constraints, cap 35, weighted score, compare | US-12.x |
| C17 | CI and Replay Harness | Pinned CI, coverage, audit, rule replay | US-10.3 |
| C18 | Fixtures and Demo Scorer | Forms, applicants, messages, postings, SC1-SC7 report | US-10.1, 10.2, 10.4, 10.5 |
| C13, C16 | Rule Sync, Shared Rule Library | Removed in version 2 | - |

## MCP Tools (29)
Profile and preferences: `register_document`, `get_document`, `save_profile_records`, `confirm_profile_section`, `get_profile`, `set_sensitive_answer`, `save_questionnaire`, `get_questionnaire`.
Filling: `plan_fill`, `find_answers`, `check_answer`, `save_answer`, `record_fill_result`, `undo_page`, `close_run`, `propose_rule`, `list_rules`.
Tracking: `log_application`, `confirm_submission`, `update_status`, `list_pending_updates`, `resolve_pending_update`, `undo_last_change`, `list_applications`, `export_tracker_csv`.
Ranking: `save_job`, `score_job`, `compare_jobs`.
Data: `delete_my_data`.

## Services
S1 Profile and Preferences Setup, S2 Fill (per page), S3 Fill-Sheet Fallback, S4 Teach a Fix, S7 Data Deletion, S8 Demo Scoring, S9 Tracking, S10 Status Update, S11 Ranking (S5 and S6 removed). Steps in `services.md`.

## Dependencies
Skills -> C4 only; C4 -> domain components; domain components -> C14 only for data; C23 -> C22 (set Ranked), never the reverse; pure logic in C10, C12, C22 and C23 is shared with tests; no cycles. Matrix and data-flow diagrams in `component-dependency.md`.

## Completeness Check
- Every story US-1.1 to US-12.4 (39 stories) maps to at least one component and one service.
- Consistent with `tech-env.md` (version 2): five skills, `node:sqlite`, Zod, esbuild single file, read-only Gmail as an option, no server network access, prohibited libraries respected.
- The Chrome plus MCP risk has a designed alternate path (S3), and tracking and ranking do not depend on the browser.

## Extension Compliance Summary (Application Design, version 2)

| Rule | Status | Note |
| --- | --- | --- |
| SECURITY-01 | Compliant (adapted) | Local SQLite in the user-only plugin data folder on an OS-encrypted disk; no server network traffic |
| SECURITY-03 | Compliant (adapted) | C15 allowlisted logs; no message or posting text |
| SECURITY-05 | Compliant | Zod bounds on all 29 tools (for example posting text up to 20,000 characters); parameterized queries |
| SECURITY-08 | Compliant | Tools act only on the local student's own data; destructive or status-changing tools require explicit confirmation, a pending queue, or an undo entry |
| SECURITY-09 | Compliant | No default credentials; generic error messages |
| SECURITY-10 | Compliant | Lock file, `npm ci`, `npm audit`, pinned CI in C17 |
| SECURITY-11 | Compliant | Security-critical logic isolated (C9, C12, C14, C22, C23); misuse cases: prompt injection through pages, postings and messages; submit and Submitted boundaries; a message cannot change more than one application |
| SECURITY-13 | Compliant | No downloaded code or rules; rules interpreted as five fixed step types; CI changes reviewed |
| SECURITY-15 | Compliant | Structured errors, global handler, fail-closed state machine and transitions, transactions with rollback |
| SECURITY-02, 04, 06, 07, 12, 14 | N/A | No intermediaries, HTML endpoints, cloud IAM, network configuration, authentication or deployed monitoring |
| PBT-09 | Compliant | fast-check with Vitest |
| PBT-02, 03, 07, 08 | Planned | Properties for C10, C12, C22 (`canTransition`, `displayStatus`, CSV round-trip) and C23 (score in 0-100, cap at 35) are enforced in Functional Design and Code Generation |
