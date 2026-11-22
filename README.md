# Job Application Assistant
INFO 5940 · Fall 2026 · README · Team review draft · 22 November 2026

An LLM-assisted Chrome extension for filling job applications, correcting résumé parsing errors, and reusing verified form repairs.

## Team and tools
| Name | NetID |
| --- | --- |
| Tifany Martinez | tm783 |
| Liaba Gondal | lg695 |
| Eesha Kamran | ek843 |
| Chen Chen | cc3254 |
| Mikey He | ch2373 |

Development uses OpenAI Codex with AI-DLC v1.0.1. The existing rules are in AGENTS.md and .aidlc-rule-details/. The planned product provider is OpenAI, with GPT-4.1 mini as a provisional API model for team review. Final selection will use tests of extraction, field interpretation, repair proposals, latency, and cost.

## Problem and rationale for using LLMs
Students applying for internships and early-career jobs repeatedly enter contact details, education, employment, and preferences. Uploading a résumé often leaves fields empty or assigns facts to the wrong records. A student may have to search for the correct university, reselect a degree, repair dates, and repeat the same work on the next application. Our Chrome extension uses a student-confirmed profile to fill and correct supported forms in the page the student already has open. When a control fails, the student can demonstrate the correction. The product aims to turn that demonstration into a tested repair that can help the same student again and, after review, other applicants.

An LLM is appropriate because résumés, field labels, and application questions express the same concepts in different ways. It can turn résumé text into separate education and employment records, interpret a field using its surrounding section, and match an open question to a relevant approved answer. It can also interpret a student's demonstrated correction and propose a reusable sequence with profile references in place of personal values. Ordinary code executes allowed actions, validates results, and preserves edits. General browser assistants already support form filling and reusable workflows. Our product hypothesis is that a focused, reviewed collection of form repairs will reduce repeated intervention. This benefit remains to be demonstrated.

## Planned LLM integration
A bounded tool-using workflow will inspect the current form, retrieve relevant confirmed profile fields or approved answers, propose mappings, and check the result of each operation. Available operations include inspecting fields, searching listed options, filling a field, reading its value back, and moving to an ordinary next step. Code enforces the allowed actions and pauses for unknown answers or unsupported controls. A second use of the model converts a short user demonstration into a candidate repair. Repairs are tested with synthetic profiles and reviewed before shared use. The hosted API receives only the information required for the task, which can include résumé text during setup. Stored profiles and answers remain local. Students approve information before reuse and control whether a sanitized repair is contributed.

## Project scope and feasibility
By December 16, the local prototype will provide a confirmed profile, one-click filling and correction, supported multipage navigation, scoped reuse of saved answers, demonstration-based repair, and a small reviewed repair library. It will cover text inputs, selects, radio buttons, searchable school controls, and repeated education or employment sections in three controlled English-language application fixtures, including single-page and multipage flows. Students paste new open-question answers and choose whether to save them for this application, this company, or general use. The extension preserves subsequent manual edits and provides a review and undo view. Students handle agreements, signatures, and final submission. Gmail tracking, job ranking, interview preparation, and generating new company-specific answers are outside this MVP.

The main technical risk is operating dynamic controls and proving that a field survives page transitions. Construction will begin with the extension and a difficult school selector, then add the profile, repeated records, multipage verification, scoped answers, and learned repairs. Acceptance testing will use three synthetic applicants, at least 60 labeled field decisions, and two repair demonstrations that transfer to a second applicant. A changed-control test must disable an incompatible repair. The course demo runs locally with a hosted model API. A reviewed, versioned repair package can be imported by another local installation, making sharing demonstrable without a public service. Real-site coverage will be documented separately from fixture results. The team will record completion time and manual interventions against manual filling without claiming an unmeasured speed advantage.

## Collaboration plan
We meet online every two to three days and schedule the next meeting at the end of each session. We communicate through WhatsApp between meetings and dedicate a full team meeting to the problem and LLM rationale. The proposed task plan uses a shared checklist, with an owner and reviewer agreed for each unit during Inception. Work is divided across profile and answers, form execution and navigation, repair learning and validation, and review controls and integration testing. One driver runs Codex in a shared session while everyone discusses the problem, answers questions, reviews artifacts, and approves each gate. Technical members lead Application Design. The team keeps audit.md and aidlc-state.md complete, resolves changes through AI-DLC, and stops before Construction after approving Units Generation. During Construction, feature branches and pull requests receive review from another teammate. These coordination details will be ratified with the input documents before Inception begins.

## Milestones and repository plan
| Milestone | Required outcome |
| --- | --- |
| November 24, 2026 | Review input documents and complete one shared AI-DLC Inception workflow through Units Generation. |
| December 16, 2026 | Complete Construction for every unit and submit the product demo, presentation, repository, and peer evaluation. |

Proposed input paths are README.md, vision.md, and tech-env.md. Generated Inception artifacts belong in aidlc-docs/inception/, with audit.md and aidlc-state.md in aidlc-docs/. Application code belongs at the repository root. All committed examples and demo materials use synthetic data. This README and the vision are drafts for review. A new technical environment document and the Inception artifacts are separate remaining deliverables.

[Download the Word version](docs/README_Job_Application_Assistant.docx)

[Read the vision document](vision.md)
