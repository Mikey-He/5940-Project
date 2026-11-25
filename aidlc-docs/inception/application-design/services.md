# Services and Orchestration

**Version 2** (after the change request): adds S9 Tracking, S10 Status Update and S11 Ranking; removes S5 Rule Review and Publication and S6 Rule Sync (shared fix library out of scope); S2, S4, S7 and S8 updated.

Orchestration follows Q2: **skills orchestrate the conversation and browser actions**, and the **server enforces order and safety** through the Run Manager state machine, the Tracker Service status rules, the Fit Scorer constraints, and structured error codes.

## S1 - Profile and Preferences Setup
**Driver**: C1. **Server**: C7, C6, C21, C9, C14. **Stories**: US-1.1 to US-1.5, US-3.1

1. Skill asks for the résumé (drag and drop, or typed path as fallback); `register_document(path, "resume")`.
2. Claude extracts contact, degrees and jobs; `save_profile_records` stores them unconfirmed.
3. Student edits and confirms each section; `confirm_profile_section`.
4. Skill asks the preference questionnaire; `save_questionnaire` (C21 validates weights and the No response cutoff).
5. Optional: cover letter (`register_document`), fixed sensitive answers (`set_sensitive_answer`).

## S2 - Fill Service (per page)
**Driver**: C2. **Server**: C5, C10, C9, C11, C12, C7, C8, C22. **Stories**: US-2.1 to US-2.5, US-3.2, US-4.1 to US-4.3, US-7.1, US-11.1

| From | Event | To |
| --- | --- | --- |
| planned | batch fill done, originals sent | filled |
| filled | `record_fill_result` | verified |
| verified | `close_run` (next_page or done) | closed |
| any state | `close_run` (abandoned) | closed |

1. Claude reads the page once and lists fields.
2. `plan_fill` starts a page run and returns one action per field (`fill`, `rule`, `sensitive`, `upload`, `answer`, `ask_claude`, `skip`).
3. Claude handles `ask_claude` fields only when the label clearly maps to one confirmed fact.
4. Uploads use `get_document` (hash check). Open questions use `find_answers`, a draft if needed, `check_answer`, and the student's approval.
5. Claude reads originals, fills in one batch, reads every value back; `record_fill_result` verifies, counts rule outcomes, disables failing rules, returns the review list.
6. Undo on request through `undo_page`.
7. Multipage: after approval, Claude clicks "Next" (never a submit control), `close_run(next_page)`, repeat.
8. At the end: `close_run(done)` and `log_application` (status In progress, or the existing Ranked entry moves to In progress). The student submits personally.

## S3 - Fill-Sheet Fallback
**Driver**: C2 in fill-sheet mode. **Stories**: US-9.4
The student pastes or describes the fields; `plan_fill` runs as in S2; Claude prints a table (field, value, source, flag) marked "not verified". Tracking (S9, S10) and ranking (S11) are unaffected.

## S4 - Teach a Fix
**Driver**: C3. **Server**: C11, C12. **Stories**: US-5.1, US-5.2, US-9.2
1. A control fails in S2; the student demonstrates the steps.
2. Claude describes them with the five step types and profile values; `propose_rule` validates, fingerprints, sanitizes and scans, then stores a personal rule.
3. Later runs use the rule through S2; a read-back mismatch disables it and flags the field.

## S5 - Rule Review and Publication - REMOVED in version 2
## S6 - Rule Sync - REMOVED in version 2

## S7 - Data Deletion
**Driver**: C1. **Server**: C14. **Stories**: US-8.1
The skill states what will be deleted and asks for confirmation; `delete_my_data({ confirm: true })` deletes profile, answers, sensitive answers, questionnaire, rules, run history, jobs, rankings, applications, status history and pending updates in one transaction. Files on disk are not touched.

## S8 - Demo Scoring
**Driver**: team before the demo. **Components**: C18, C15 logs, tracker and ranking results. **Stories**: US-10.4
Run the three applicants through the three forms (SC1, SC2, SC4, SC5), scan rules, history and logs (SC3), replay the 20 labeled messages (SC6) and score the 10 postings against two questionnaires (SC7); produce one report.

## S9 - Tracking Service (new)
**Driver**: C19 (and C2 for automatic logging). **Server**: C22, C21, C14. **Stories**: US-11.1, US-11.2, US-11.5, US-11.6

1. Automatic: S2 step 8 calls `log_application`. Manual: the student says "add my Globex application"; C19 calls `log_application`. Duplicate links reuse the entry.
2. Submission: the student says they submitted; `confirm_submission` is the only path to Submitted.
3. View: `list_applications` with an optional filter; C22 computes No response from the questionnaire cutoff at read time; Claude renders the table and timelines.
4. Export: `export_tracker_csv` to a path the student chooses (never inside the repository).

## S10 - Status Update Service (new)
**Driver**: C19. **Server**: C22. **Stories**: US-11.3, US-11.4, US-11.7

Decision order inside `update_status` (C22):

1. Duplicate message ID: return `DUPLICATE_IGNORED`, no change.
2. Transition not allowed (Submitted without the student, or an automatic backwards move): return `STATUS_TRANSITION_INVALID`.
3. High confidence, exactly one matching application, and no conflict with an earlier student edit: apply, write status history and an undo entry.
4. Otherwise: store a pending proposal for the student.

1. The student states a reply or pastes an employer message (stretch goal: asks Claude to check Gmail through the read-only connector).
2. Claude classifies it and calls `update_status`; message text is data and cannot widen the action beyond one proposal.
3. `list_pending_updates` and `resolve_pending_update` let the student approve or reject; `undo_last_change` reverts the latest change.

## S11 - Ranking Service (new)
**Driver**: C20. **Server**: C23, C21, C22. **Stories**: US-12.1 to US-12.4

1. `get_questionnaire`; if missing, the skill asks the student to complete it (S1 step 4).
2. `save_job` from the open page or pasted text (no duplicates by link; no job-board searching).
3. Claude extracts facts exactly as stated (or null) and judges skills, interest and nice-to-have fit with quotes.
4. `score_job`: C23 rejects missing evidence, checks hard constraints, lists unknowns, caps deal-breakers at 35, applies weights, saves the ranking, and asks C22 to create or keep the tracker entry at Ranked (never moving it backwards).
5. Claude explains the score; `compare_jobs` shows saved jobs side by side, highest first.
