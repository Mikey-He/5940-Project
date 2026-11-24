# Job Application Assistant

INFO 5940 · Fall 2026 · A Claude plugin that fills job applications from a student's confirmed profile and learns reusable fixes for hard form controls.

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

Students applying for internships and early-career jobs type the same facts into every application. Résumé parsers leave fields empty or put a degree under the wrong school, and searchable dropdowns for universities often reject pasted text. Each student fixes the same control on every form, and every other student fixes it again. Autofill extensions stop at unusual controls, while a general browser agent can operate them but is slow and starts from zero each time. An LLM fits because every input here is unstructured language. "Institution" and "Where did you study?" both mean the school field, and only a language model can read a label in context and pick the right profile fact. The same model can draft an answer to an open question from the job description within its word limit. It can also watch a student fix a control and describe the fix as general steps, which ordinary code then checks and replays the same way every time.

## Planned LLM integration

The product is a Claude Code plugin with a true agentic pattern. Three skills tell Claude how to run each workflow. The first sets up a profile and the second fills an application. The third teaches a fix. Claude in Chrome lets Claude read and operate the employer's page in the student's own browser, and a local MCP server we write in TypeScript provides the tools. When a student says "fill this application", Claude reads the page once and sends the field labels to `plan_fill`. Code matches known fields to the confirmed profile and applies approved rules for difficult controls. Only the fields code cannot place go back to Claude. Claude fills the page in one batch and reads every value back, then reports through `record_fill_result`. For open questions, `find_answers` returns saved answers, and Claude drafts a new one only when nothing fits and the student approves it. When a control fails, the student demonstrates the fix once and `propose_rule` turns it into a rule with profile references in place of personal values.

The LLM is Claude, used through each student's own Claude plan, and we develop and test with Claude Sonnet 5.5. Claude sees the page text and the profile fields that page needs, because it types them. Profiles and answers are stored only on the student's laptop. Shared rules hold control steps and references such as `{{profile.education[0].school}}`, never a student's values.

## Project scope and feasibility

By December 16 the plugin will work on three fixture application forms we build. It turns a résumé PDF into a confirmed profile, with the file given by drag and drop or by its path, and registers a cover letter the same way. It fills single-page and multipage forms, including a searchable school field and repeated education entries, and uploads the registered files wherever a form asks for one. It reuses saved answers to open questions and drafts new ones with the student's approval. It learns a fix from one demonstration, and it shares an approved fix with other installs through a public rule folder in this repository once the fix passes schema checks and replay tests and a teammate approves it. Each run ends with a review list and an undo log, and the student always submits personally. Job tracking stays out of this MVP. So do job ranking and interview preparation.

The riskiest dependency is running Claude in Chrome and our MCP server in one session, so a teammate tests it with a hello-world plugin before Construction. If it fails, filling falls back to a field-by-field fill sheet in chat and the rest still works. The demo test is concrete. Three synthetic applicants complete the three fixture forms, covering at least 60 predefined field decisions. Applicant A teaches two fixes, and applicant B must then need fewer corrections on the same controls. Real employer sites are tried separately and reported as observations.

## Collaboration plan

We meet online every two to three days and set the next meeting at the end of each session. We held one full team meeting on the problem and the LLM rationale before starting AI-DLC. Between meetings we talk on WhatsApp and track tasks in GitHub Issues. Inception runs as one shared AI-DLC workflow in which Mikey He drives Claude Code and commits while the whole team answers questions and reviews each artifact. Mikey He and Eesha Kamran lead Application Design. Tifany Martinez leads the product questions in Requirements and User Stories together with Chen Chen and Liaba Gondal. We keep `aidlc-docs/audit.md` and `aidlc-docs/aidlc-state.md` complete and fix problems through the workflow rather than by hand. During Construction each unit of work gets one owner and one reviewer, and every change goes through a pull request reviewed by another teammate. The work splits into four streams.

- Profile and answer store
- Fill planner and skills
- Rule learning and validation
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
