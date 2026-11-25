# Job Application Assistant

INFO 5940 · Fall 2026 · A Claude plugin that helps students apply, track and rank: it fills job applications from a confirmed profile and learns fixes for hard form controls, keeps one tracker of every application and its status, and scores how well each job posting fits.

## Team and tools

| Name | NetID |
| --- | --- |
| Tifany Martinez | tm783 |
| Liaba Gondal | lg695 |
| Eesha Kamran | ek843 |
| Chen Chen | cc3254 |
| Mikey He | ch2373 |

- We follow AI-DLC v1.0.1 as our development methodology.
- Our AI coding agent is Claude Code. The AI-DLC v1.0.1 rules are committed as `CLAUDE.md` and `.aidlc-rule-details/`. `AGENTS.md` holds the same core rules so a teammate on OpenAI Codex can also follow the workflow.
- The LLM inside the product is Claude Sonnet 5.5, used through each student's own Claude plan. The plugin runs inside Claude Code, so Claude is both the model and the agent.

## Problem and rationale for using LLMs

Students applying for internships and early-career jobs repeat the same work three ways. They type the same facts into every application, while résumé parsers leave fields empty or put a degree under the wrong school, and searchable dropdowns for universities often reject pasted text. After applying, their applications scatter across portals and inboxes, so they lose track of where they applied and who replied. Before applying, they face dozens of postings with no quick, consistent way to judge which ones fit their skills, location and pay needs. Autofill extensions stop at unusual controls, spreadsheets go stale, and job boards rank for the employer, not the student.

An LLM fits because every input here is unstructured language. "Institution" and "Where did you study?" both mean the school field, and only a language model can read a label in context and pick the right profile fact. It can draft an answer to an open question within its word limit and turn a student's demonstrated fix into general steps. It can read an employer's email and tell an interview invitation from a rejection, and it can read a posting and judge how well its requirements match the student, quoting the evidence. In every case ordinary code then checks the result: it matches known fields, applies status rules, enforces hard constraints and computes the final score the same way every time.

## Planned LLM integration

The product is a Claude Code plugin with a true agentic pattern. Five skills tell Claude how to run each workflow: set up a profile and preferences, fill an application, teach a fix, track applications, and rank a job. Claude in Chrome lets Claude read and operate the employer's page in the student's own browser, and a local MCP server we write in TypeScript provides the tools. When a student says "fill this application", Claude reads the page once and sends the field labels to `plan_fill`. Code matches known fields to the confirmed profile and applies saved rules for difficult controls. Only the fields code cannot place go back to Claude. Claude fills the page in one batch, reads every value back, reports through `record_fill_result`, and logs the application in the tracker. For open questions, `find_answers` returns saved answers, and Claude drafts a new one only when nothing fits and the student approves it. When a control fails, the student demonstrates the fix once and `propose_rule` turns it into a rule with profile references in place of personal values.

For tracking, the student tells Claude about a reply or pastes an employer message. Claude classifies it and calls `update_status`, which applies a clear match with an undo entry and holds an uncertain one for the student to confirm. Reading Gmail through Claude's read-only connector is a stretch goal. For ranking, Claude extracts a posting's location, arrangement and pay and judges skills, interest and nice-to-have fit with quotes, and `score_job` checks the student's hard constraints and computes a 0–100 score.

The LLM is Claude, used through each student's own Claude plan, and we develop and test with Claude Sonnet 5.5. Claude sees the page text, pasted messages and postings, and the profile fields the current task needs. Profiles, answers, the tracker and rankings are stored only on the student's laptop.

## Project scope and feasibility

By December 16 the plugin will cover apply, track and rank on synthetic data. It turns a résumé PDF into a confirmed profile and a preference questionnaire, with the file given by drag and drop or by its path, and registers a cover letter the same way. It fills single-page and multipage forms on three fixture forms we build, including a searchable school field and repeated education entries, and uploads the registered files wherever a form asks for one. It reuses saved answers to open questions and drafts new ones with the student's approval. It learns a fix from one demonstration and reuses it on the same kind of control. Each run ends with a review list and an undo log, and the student always submits personally. Every application lands in a tracker with statuses from Ranked to Offer or Rejected, No response computed after a cutoff, a table view in chat and CSV export. Any saved posting gets a 0–100 fit score with evidence, deal-breakers capped by code, and side-by-side comparison. Sharing fixes across students, job-board searching and interview preparation stay out of this MVP.

The riskiest dependency is running Claude in Chrome and our MCP server in one session, so a teammate tests it with a hello-world plugin before Construction. If it fails, filling falls back to a field-by-field fill sheet in chat and tracking and ranking still work, because they need no browser control. The demo tests are concrete. Three synthetic applicants complete the three fixture forms, covering at least 60 predefined field decisions. At least 20 labeled synthetic employer messages measure status accuracy, and at least 10 synthetic postings are ranked against two questionnaires, with the team checking the top-3 orderings. Real employer sites are tried separately and reported as observations.

## Collaboration plan

We meet online every two to three days and set the next meeting at the end of each session. We held one full team meeting on the problem and the LLM rationale before starting AI-DLC. Between meetings we talk on WhatsApp and track tasks in GitHub Issues. Inception runs as one shared AI-DLC workflow in which Mikey He drives Claude Code and commits while the whole team answers questions and reviews each artifact. Mikey He and Eesha Kamran lead Application Design. Tifany Martinez leads the product questions in Requirements and User Stories together with Chen Chen and Liaba Gondal. We keep `aidlc-docs/audit.md` and `aidlc-docs/aidlc-state.md` complete and fix problems through the workflow rather than by hand. During Construction each unit of work gets one owner and one reviewer, and every change goes through a pull request reviewed by another teammate. The work splits into five streams.

- Profile, preferences and answer store
- Fill planner and skills
- Rule learning
- Application tracking and job ranking
- Fixtures and tests

## Repository contents

| Path | What it is |
| --- | --- |
| `README.md` | This overview |
| `vision.md` | The product and its users. It also sets the MVP scope and lists open questions plus what is out of scope |
| `tech-env.md` | How the plugin is built and how it is tested. It also holds the security rules. Example code and prohibited libraries are there too |
| `CLAUDE.md`, `AGENTS.md`, `.aidlc-rule-details/` | AI-DLC v1.0.1 rule files for Claude Code (and Codex) |
| `aidlc-docs/` | Everything AI-DLC generates, including `inception/` and the two record files |

Plugin code is added during Construction under `plugins/job-application-assistant/`. All examples and test data are synthetic. No real résumés or personal data are committed.
