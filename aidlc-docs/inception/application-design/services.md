# Services and Orchestration

Orchestration is split as decided in Q2: **skills orchestrate the conversation and browser actions**, and the **server enforces order and safety** through the Run Manager state machine and structured error codes. Every service below names the skill that drives it and the server components it uses.

## S1 - Profile Setup Service
**Driver**: C1 Set-Up-Profile Skill. **Server**: C7, C6, C14. **Stories**: US-1.1 to US-1.4

1. Skill asks for the résumé (drag and drop, or typed path as fallback).
2. `register_document(path, "resume")` - C7 checks type and size, stores path, size, SHA-256.
3. Claude reads the PDF and extracts contact, degrees and jobs as separate records.
4. `save_profile_records` - C6 validates, deduplicates, stores as unconfirmed.
5. Skill shows each section; student edits and confirms; `confirm_profile_section` per section.
6. Optional: `register_document(path, "cover_letter")`; `set_sensitive_answer` per category the student chooses.

## S2 - Fill Service (per page)
**Driver**: C2 Fill-Application Skill. **Server**: C5, C10, C9, C11, C12, C7, C8. **Stories**: US-2.1 to US-2.5, US-3.2, US-4.1 to US-4.3, US-7.1

```text
 planned  --(batch fill done, originals sent)-->  filled
 filled   --(record_fill_result)-->               verified
 verified --(close_run: next_page/done)-->        closed
 any      --(close_run: abandoned)-->             closed
```

1. Claude reads the page once and lists fields (label, type, options, element reference, control structure).
2. `plan_fill` - C5 starts a page run (`planned`); C10 returns one action per field:
   - `fill` from confirmed profile; `rule` with resolved steps (control fingerprint match); `sensitive` via C9 (exact match or blank); `upload`; `answer`; `ask_claude`; `skip`.
3. Claude handles `ask_claude` fields only when the label clearly maps to one confirmed fact; otherwise leaves them blank.
4. For `upload`: `get_document` (C7 verifies hash) then upload through Claude in Chrome.
5. For `answer`: `find_answers`; if none fits, Claude drafts; `check_answer` counts words; the student approves or edits; optionally `save_answer`.
6. Claude reads current values (originals), fills everything in one batch, then reads every value back.
7. `record_fill_result` - C5 moves to `verified`, keeps originals in memory for undo; C11 notes clean read-backs or disables failing rules; returns the review list.
8. Skill shows the review list. On "undo": `undo_page` returns restore actions, Claude applies them.
9. Multipage: after the student approves, Claude clicks "Next" (never a submit control), calls `close_run(next_page)`, and repeats from step 1 with the same `runId`.
10. At the end: `close_run(done)`; the student submits personally.

**Guards**: `record_fill_result` before a plan returns `RUN_STATE_INVALID`; login pages and CAPTCHAs pause the skill; page text is data, not instructions.

## S3 - Fill-Sheet Fallback Service
**Driver**: C2 in fill-sheet mode. **Server**: same as S2 without browser steps. **Stories**: US-9.4

1. Skill detects that Claude in Chrome tools are unavailable.
2. Student pastes or describes the form fields; Claude structures them as `Field[]`.
3. `plan_fill` as in S2; sensitive and answer rules apply unchanged.
4. Skill prints a table (field, value, source, flag) for the student to copy. No read-back; every row is marked "not verified".

## S4 - Teach and Share Service
**Driver**: C3 Teach-a-Fix Skill. **Server**: C11, C12. **Stories**: US-5.1, US-5.2, US-6.1, US-9.2

1. A control fails in S2; the student demonstrates the working steps.
2. Claude describes the steps with the five step types and profile values.
3. `propose_rule` - C12 validates the schema, computes the control fingerprint, sanitizes values into references; C11 stores a personal rule.
4. Later runs use the rule through S2; each clean read-back is counted.
5. `list_share_candidates` shows rules with at least one clean later read-back.
6. On consent, `export_rule_candidate` - C12 personal-data scan; the JSON is shown and written for a pull request.
7. On a failed shared rule, `draft_rule_issue` returns a title and body with label, fingerprint and failing step only.

## S5 - Rule Review and Publication Service
**Driver**: P4 reviewer on GitHub. **Components**: C16, C17, C12. **Stories**: US-6.2, US-6.4, US-10.3

1. A pull request adds or removes a rule file and updates `index.json` (with SHA-256).
2. CI (C17): `npm ci`, unit and property tests with coverage, `npm audit`, schema and personal-data checks, Playwright replay for applicants A, B, C on the three forms.
3. Branch protection requires all checks green and one approval from a non-author within 48 hours.
4. Withdrawal is a pull request that removes the rule from the index.

## S6 - Rule Sync Service
**Driver**: server startup. **Server**: C13, C12, C14. **Stories**: US-6.3, US-9.3

1. On session start the server calls `sync_rules` internally (and the tool can be called on demand).
2. Fetch `index.json` over HTTPS; download new or changed files; verify SHA-256 and schema; replace the cache; drop removed rules.
3. Offline: keep the cache and return `offline: true`; the skill tells the student once.

## S7 - Data Deletion Service
**Driver**: C1. **Server**: C14. **Stories**: US-8.1

1. Student asks to delete data; the skill states what will be deleted and asks for confirmation.
2. `delete_my_data({ confirm: true })` deletes records, documents, answers, sensitive answers, personal rules, rule cache and run history in one transaction. Files on disk are not touched.

## S8 - Demo Scoring Service
**Driver**: team before the demo. **Components**: C18, C15 logs. **Stories**: US-10.4

1. Run applicants A, B, C through the three fixture forms; collect run logs and read-backs.
2. Score against the answer key (SC1); compare A and B corrections on taught controls (SC2); scan shared rules for synthetic values (SC3); run the changed-control fixture (SC4); record the SC5 draft check.
