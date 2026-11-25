# Application Design Plan

**Inputs**: `requirements.md`, `stories.md`, `personas.md`, `execution-plan.md`, `tech-env.md`
**Scope**: Component boundaries, method signatures, service orchestration and dependencies for the Claude Code plugin, the local MCP server, the shared rule library and the test harness. Detailed business rules are left to Functional Design in Construction.

## Questions

Please answer each question by putting the letter after the `[Answer]:` tag. If no option fits, choose the last option (Other) and describe your choice.

## Question 1
How should the MCP server code be organized?

A) By domain only: one module per area (profile, documents, answers, planner, rules, sanitizer, sync), each talking to SQLite directly

B) Strict layers: tools call services, services call repositories

C) Thin tool handlers (Zod validation and error mapping only) that call domain modules, which use one repository module as the only code touching `node:sqlite`

X) Other (please describe after [Answer]: tag below)

[Answer]: C
Rationale: keeps validation at the edge (SECURITY-05), keeps all SQL parameterized in one module, and lets domain modules be tested as pure functions.

## Question 2
Where does workflow orchestration live?

A) Entirely in the skills: Claude follows the SKILL.md steps and the tools are independent primitives

B) In a server-side orchestrator tool (for example `run_fill`) that drives the whole fill

C) Skills orchestrate the conversation and browser actions, while the server keeps a small run state machine (planned, filled, verified, closed) and refuses out-of-order calls, so safety rules hold even if Claude skips a step

X) Other (please describe after [Answer]: tag below)

[Answer]: C
Rationale: Claude stays in charge of language and browser steps, but the server enforces order (no read-back before a plan, no closing before read-back), so a skipped step fails closed (SECURITY-15).

## Question 3
How should code match a field label to a profile fact?

A) Normalized label against a synonym table only; anything else goes to Claude

B) Synonym table plus fuzzy string matching above a similarity threshold

C) Embedding similarity

X) Other (please describe after [Answer]: tag below)

[Answer]: A
Rationale: matches "flagged, never guessed" and the example in tech-env.md; fuzzy matches are exactly the ambiguous cases Claude should handle with a visible "decided by Claude" mark.

## Question 4
What does a rule's fingerprint identify?

A) The page (URL host and path)

B) The control: its role and type, normalized label, and structural features such as option count and input pattern, independent of the URL

C) Both: a page fingerprint for run history, and a control fingerprint for matching rules, so a rule taught on one form applies to the same kind of control on another

X) Other (please describe after [Answer]: tag below)

[Answer]: C
Rationale: US-5.2 needs a taught fix to work on the same kind of control on another form, while run history still needs to know which page it was on.

## Question 5
How are a page's original values kept for undo, given that history must not store values?

A) In the run history table, then deleted when the student submits

B) In memory inside the MCP server for the active run only, discarded when the run closes, the next page starts, or the session ends

C) Not stored; undo clears every field the plugin touched

X) Other (please describe after [Answer]: tag below)

[Answer]: B
Rationale: undo needs the old values only while the student is on that page; keeping them in memory honors NFR-2.4 (no values in history).

## Question 6
Who decides that a field is sensitive?

A) Code only, with a fixed list of question patterns per category

B) Claude only

C) Code decides with a fixed pattern list; Claude may additionally mark a field as sensitive (never the reverse), and any field marked sensitive is handled only by exact-match fixed answers

X) Other (please describe after [Answer]: tag below)

[Answer]: C
Rationale: a code pattern list is testable and deterministic, and letting Claude only make a field more restricted can never cause a sensitive answer to be guessed.

## Question 7
How should tools report errors?

A) Throw exceptions and let the SDK report them

B) Every tool returns a structured result with `ok`, a stable error `code` and a generic `message`; a top-level handler catches anything unexpected and returns a generic error, with details only in the local log

X) Other (please describe after [Answer]: tag below)

[Answer]: B
Rationale: stable codes let skills react correctly (for example FILE_CHANGED triggers US-9.1), and generic messages with local-only details satisfy SECURITY-15.

## Question 8
How is rule logic shared between the runtime plugin and the CI replay tests?

A) Separate code in each place

B) One shared core module (rule schema, reference resolution, sanitizer) used by both the MCP server and the Playwright test executor; only the step executor differs (Claude in Chrome at runtime, Playwright in CI)

X) Other (please describe after [Answer]: tag below)

[Answer]: B
Rationale: CI replay then tests the same schema, reference resolution and sanitizer that students run, so a rule that passes CI behaves the same at runtime.

## Question 9
How is the fill-sheet fallback (Chrome tools unavailable, US-9.4) designed?

A) A separate fourth skill

B) A mode of the fill-application skill that uses the same `plan_fill` and safety rules, with fields described by the student instead of read from the browser, and output as a table in chat

X) Other (please describe after [Answer]: tag below)

[Answer]: B
Rationale: the fallback reuses planning and safety rules instead of duplicating them, and a fourth skill would add surface without new value.

## Question 10
Where do logs go?

A) Standard output

B) Structured JSON lines written to a log file in the plugin data folder and to standard error, never standard output (stdout carries the MCP protocol)

X) Other (please describe after [Answer]: tag below)

[Answer]: B
Rationale: stdout is reserved for the MCP stdio protocol, and a local JSON-lines file gives structured logs with run IDs and no values (SECURITY-03 adapted).

## Question 11
How should components be tested below the Playwright level?

A) Unit tests only

B) Unit and property tests on domain modules with an in-memory SQLite database, plus in-process integration tests that call each MCP tool through the SDK client

X) Other (please describe after [Answer]: tag below)

[Answer]: B
Rationale: in-memory SQLite keeps tests fast and isolated, and SDK-level integration tests prove each tool's Zod contract and error codes.

## Execution Checklist

- [x] 1. Validate all answers and resolve any ambiguity
- [x] 2. Generate `components.md`: component names, purposes, responsibilities and interfaces
- [x] 3. Generate `component-methods.md`: method signatures with input and output types (business rules deferred to Functional Design)
- [x] 4. Generate `services.md`: service definitions and orchestration for setup, fill, teach, share, sync and delete
- [x] 5. Generate `component-dependency.md`: dependency matrix, communication patterns and data flow diagrams
- [x] 6. Generate `application-design.md` consolidating the four documents
- [x] 7. Validate completeness against stories US-1.1 to US-10.4 and consistency with `tech-env.md`
- [x] 8. Check extension compliance (Security Baseline, PBT Partial)
