# Story Generation Plan

**Role**: Product owner
**Inputs**: `aidlc-docs/inception/requirements/requirements.md`, `vision.md`, `tech-env.md`, `aidlc-docs/inception/plans/user-stories-assessment.md`
**Outputs**: `aidlc-docs/inception/user-stories/personas.md`, `aidlc-docs/inception/user-stories/stories.md`

## Story Breakdown Approaches Considered

| Approach | How stories are organized | Benefit | Trade-off for this project |
| --- | --- | --- | --- |
| User Journey-Based | Follow the student's path: set up, fill, answer, fix, share, review | Mirrors how the demo is run | Shared-rule review and data deletion do not sit on the main journey |
| Feature-Based | One group per vision feature F1-F7 | Maps directly to requirements and to the four work streams | Can hide the end-to-end flow |
| Persona-Based | Stories grouped by Maya, Daniel, reviewer | Makes persona needs explicit | The same capability (fill) repeats across personas |
| Domain-Based | Profile, filling, answers, rules | Close to the code modules | Too technical for acceptance testing |
| Epic-Based | Epics with child stories | Clear hierarchy for traceability | Needs a rule for what becomes an epic |

## Questions

Please answer each question by putting the letter after the `[Answer]:` tag. If no option fits, choose the last option (Other) and describe your choice.

## Question 1
How should stories be organized?

A) Feature-based: one group per vision feature F1-F7

B) User journey-based: stories in the order a student meets them

C) Epic-based hybrid: one epic per vision feature (F1-F7) plus one epic for data control and one for demo enablement, with stories inside each epic ordered along the student's journey

X) Other (please describe after [Answer]: tag below)

[Answer]: C
Rationale: epics keep traceability to F1-F7 and the requirement IDs, while journey order inside each epic keeps the end-to-end demo flow visible.

## Question 2
Which personas should be written?

A) Maya and Daniel only, as described in vision.md

B) Maya and Daniel, plus a teammate who reviews contributed rules

C) Maya and Daniel, plus an international student who must answer sponsorship and work-authorization questions, plus a teammate who reviews contributed rules

X) Other (please describe after [Answer]: tag below)

[Answer]: C
Rationale: sponsorship and work-authorization questions are the highest-risk part of the product, so an international-student persona makes those rules concrete; the reviewer persona owns the shared-library workflow.

## Question 3
How large should each story be?

A) One story per user-visible capability, small enough for one pull request with one owner and one reviewer

B) One story per vision feature (seven large stories)

C) One story per MCP tool

X) Other (please describe after [Answer]: tag below)

[Answer]: A
Rationale: matches the Construction rule of one owner and one reviewer per pull request, and keeps every story testable on its own.

## Question 4
What format should acceptance criteria use?

A) Given / When / Then scenarios only

B) A checklist of testable statements

C) Given / When / Then scenarios, plus a traceability line naming the requirement IDs (FR / NFR) and demo success criteria (SC) each story covers

X) Other (please describe after [Answer]: tag below)

[Answer]: C
Rationale: Given/When/Then scenarios become test cases directly, and the traceability line proves coverage of FR-1 to FR-9 and SC1-SC5.

## Question 5
How should failure and edge cases be covered?

A) Inside each story's acceptance criteria only

B) Inside acceptance criteria, plus separate stories for failures the student must act on (changed résumé file, read-back mismatch, GitHub unreachable, Chrome plus MCP unavailable)

C) A single "error handling" story

X) Other (please describe after [Answer]: tag below)

[Answer]: B
Rationale: these four failures each need the student to act, so they deserve their own stories; smaller edge cases stay in acceptance criteria.

## Question 6
Should team-facing work (fixture forms, synthetic applicants, CI replay of shared rules) appear as stories?

A) Yes, as enabler stories in a "demo enablement" epic, written for the teammate or the demo evaluator

B) No, leave it to Units Generation as technical work

X) Other (please describe after [Answer]: tag below)

[Answer]: A
Rationale: the fixtures, synthetic applicants and CI replay are what make the demo success criteria measurable, so they need acceptance criteria too.

## Question 7
Should each story name the personas it serves?

A) Yes, each story lists its primary persona, and personas.md lists the stories for each persona

B) No, personas are described separately

X) Other (please describe after [Answer]: tag below)

[Answer]: A
Rationale: two-way mapping makes it easy to check that each persona's needs are covered.

## Question 8
Should stories state what the plugin must never do (for example, never submit, never draft a sensitive answer)?

A) Yes, as explicit negative acceptance criteria ("Then the plugin does not ...") in the stories where they apply

B) No, keep these only in requirements.md

X) Other (please describe after [Answer]: tag below)

[Answer]: A
Rationale: the safety boundaries (never submit, never draft sensitive answers, never type unapproved text) are what the demo must prove, so they should be testable in the stories.

## Execution Checklist

- [x] 1. Validate all answers above and resolve any ambiguity
- [x] 2. Generate `personas.md`: user archetypes with background, goals, frustrations, technical comfort and the stories each persona drives
- [x] 3. Define epics and story IDs according to the approved organization
- [x] 4. Generate `stories.md` with every story in "As a ..., I want ..., so that ..." form
- [x] 5. Write acceptance criteria for every story in the approved format, including failure and negative cases
- [x] 6. Check every story against INVEST (Independent, Negotiable, Valuable, Estimable, Small, Testable)
- [x] 7. Map personas to stories in both files
- [x] 8. Verify coverage: every functional requirement FR-1 to FR-9 and every success criterion SC1-SC5 is covered by at least one story
- [x] 9. Check extension compliance (Security Baseline, PBT Partial) for the story artifacts
