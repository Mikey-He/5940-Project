# Unit of Work Plan

**Inputs**: `requirements.md`, `stories.md` (31 stories), `application-design/` (components C1-C18, services S1-S8), `execution-plan.md`, `README.md` (four work streams, one owner and one reviewer per unit)
**Context**: Greenfield. One deployable product (a Claude Code plugin with one bundled MCP server) plus repository assets (shared rules, CI, fixtures). Units are logical modules for development, not separately deployed services.

## Questions

Please answer each question by putting the letter after the `[Answer]:` tag. If no option fits, choose the last option (Other) and describe your choice.

## Question 1 - Story grouping and number of units
How should the system be split into units of work?

A) Four units matching the README work streams: profile and answer store; fill planner and skills; rule learning and validation; fixtures and tests

B) Five units: a foundation unit (plugin scaffold, tool layer skeleton, repository, logger, shared types, build, and the Chrome plus MCP spike) followed by the four README work streams

C) One unit per component (18 units)

X) Other (please describe after [Answer]: tag below)

[Answer]: B
Rationale: the foundation unit removes the biggest risk (the Chrome plus MCP spike) first and gives five units for five teammates; the other four match the README work streams.

## Question 2 - Shared contracts and integration
How do units integrate while being built in parallel?

A) Each unit defines its own types; integrate at the end

B) Contract first: the foundation unit publishes the shared TypeScript types and Zod schemas (Field, PlannedAction, Rule, ReadBack, Result and error codes) and the tool registration pattern; other units build against them and integrate through the tool layer

X) Other (please describe after [Answer]: tag below)

[Answer]: B
Rationale: shared types and Zod schemas let four units work in parallel without guessing each other's interfaces, and integration happens through the tool layer the design already defines.

## Question 3 - Where the shared Rule Core (C12) lives
The planner (fill unit) needs the rule schema and fingerprints, while the rule unit owns sanitization and the CI harness.

A) Entirely in the rule learning unit; the fill unit waits for it

B) The rule schema and fingerprint functions are part of the shared contracts in the foundation unit; sanitizer, personal-data scan and reference resolution are built in the rule learning unit

C) Entirely in the foundation unit

X) Other (please describe after [Answer]: tag below)

[Answer]: B
Rationale: the planner only needs the schema and fingerprints, so publishing them early unblocks the fill unit, while the privacy-critical sanitizer and scan stay with the rule unit that owns SC3.

## Question 4 - Build order
In what order should Construction work through the units?

A) Strictly one unit after another

B) Foundation first (including the spike); then profile and answer store, rule learning and validation, and fixtures and tests in parallel; then the fill planner and skills, which integrates everything; then Build and Test

X) Other (please describe after [Answer]: tag below)

[Answer]: B
Rationale: parallel work after the foundation fits the three weeks to December 16, and the fill unit goes last because it integrates the store, rules and fixtures.

## Question 5 - Ownership
How are owners and reviewers assigned?

A) Assign now in this document

B) Units are sized so each of the five teammates can own one; owners and reviewers are assigned at the first Construction meeting and recorded in `aidlc-state.md`, with every owner reviewing a different unit than their own

X) Other (please describe after [Answer]: tag below)

[Answer]: B
Rationale: owners are best chosen together at the first Construction meeting, and cross-review keeps the README rule that every pull request is reviewed by another teammate.

## Question 6 - Code organization
How is the code organized in the repository?

A) As in `tech-env.md`: one npm package under `plugins/job-application-assistant/` (skills, `server/src`, `server/test`, committed `server/dist/index.js`), with `rules/approved/`, `fixtures/`, `tests/replay/` and `.github/workflows/` at the repository root

B) An npm workspace with a separate package per unit

X) Other (please describe after [Answer]: tag below)

[Answer]: A
Rationale: matches tech-env.md and the single bundled server file; workspaces would add tooling without benefit for one deployable plugin.

## Question 7 - Unit definition of done
When is a unit complete in Construction?

A) Its code is merged

B) All acceptance criteria of its mapped stories pass; unit and property tests pass with coverage at least 70% for its modules; the Security Baseline compliance summary has no blocking findings; its pull request is approved by a teammate other than the owner

X) Other (please describe after [Answer]: tag below)

[Answer]: B
Rationale: ties each unit to its stories, the coverage target, the Security Baseline and the review rule, so done means demo-ready.

## Execution Checklist

- [x] 1. Validate all answers and resolve any ambiguity
- [x] 2. Generate `aidlc-docs/inception/application-design/unit-of-work.md` with unit definitions, responsibilities, components, deliverables and definition of done
- [x] 3. Document the code organization strategy (greenfield) in `unit-of-work.md`
- [x] 4. Generate `aidlc-docs/inception/application-design/unit-of-work-dependency.md` with the dependency matrix, shared contracts and build order
- [x] 5. Generate `aidlc-docs/inception/application-design/unit-of-work-story-map.md` mapping every story US-1.1 to US-10.4 to a unit
- [x] 6. Validate unit boundaries and dependencies (no cycles; every component has one owning unit, with C12 split between two units only as decided in Question 3)
- [x] 7. Ensure all 31 stories are assigned to units
- [x] 8. Check extension compliance (Security Baseline, PBT Partial)
