# Personas - Job Application Assistant

All personas are fictional. Their details are illustrative and match the synthetic test data rules (no real personal data).

---

## P1 - Maya Lopez, the high-volume internship applicant

| Attribute | Detail |
| --- | --- |
| Background | Junior, B.S. in Information Science with a minor in Economics. Applying to 40 summer internships in product and data roles this season |
| Technical comfort | Medium. Uses Claude daily, installed Claude Code for a class, comfortable typing a file path, not comfortable editing JSON |
| Devices | Windows laptop, Chrome |
| Goals | Finish each application in minutes; check every value before she submits; never type her school name into a searchable dropdown again |
| Frustrations | Résumé parsers put her minor in the major field; university dropdowns reject pasted text; repeated "add another education" sections; three-box date fields |
| What success looks like | She opens a form, says "fill this application", reviews a short list of flagged fields and submits |
| Trust boundary | Wants to see what Claude decided versus what came straight from her profile |

**Design implications**: section-level confirmation must be fast (Q9); Claude-decided fields must be labeled; hard controls (school combobox, split date, repeated sections) are the core of rule learning; the plugin must never submit.

**Primary stories**: US-1.1, US-1.2, US-1.3, US-2.1, US-2.2, US-2.3, US-2.4, US-2.5, US-5.1, US-5.2, US-6.1, US-7.1, US-7.2, US-9.1, US-9.2, US-9.4

---

## P2 - Daniel Okafor, the open-question writer

| Attribute | Detail |
| --- | --- |
| Background | First-year master's student in Data Science applying to about 25 early-career analyst and ML roles |
| Technical comfort | High. Uses Claude Code for coursework and keeps answers in a Markdown notes file |
| Devices | MacBook, Chrome |
| Goals | Reuse his best "Why this company?" and "Tell us about a project" answers, adapted to each job description and word limit |
| Frustrations | Spends 20 minutes per application trimming saved answers to new word limits; worries about reusing an answer that names the wrong company |
| What success looks like | The plugin suggests the right saved answer for this company, or drafts one within the limit, and nothing is typed until he approves it |
| Trust boundary | Never wants a draft typed without approval; wants to know when a saved answer is old |

**Design implications**: answer scope (application, company, general) prevents naming the wrong company; word limits are checked by code; old answers are marked (Q5); every answer needs explicit approval.

**Primary stories**: US-1.4, US-4.1, US-4.2, US-4.3, US-7.1, US-8.1, US-9.3

---

## P3 - Priya Raman, the international student

| Attribute | Detail |
| --- | --- |
| Background | Senior on an F-1 student visa, applying to software engineering internships that may require CPT authorization and future sponsorship |
| Technical comfort | Medium-high |
| Devices | Windows laptop, Chrome |
| Goals | Answer "Will you now or in the future require sponsorship?" and "Are you authorized to work in the US?" exactly and consistently on every form |
| Frustrations | A wrong sponsorship answer can disqualify her or misrepresent her status; autofill tools sometimes guess these questions; demographic questions she prefers to answer herself |
| What success looks like | Her fixed answers are used only on exact matches, and any sensitive question she has not set is left blank and flagged for her |
| Trust boundary | The plugin must never decide, draft or learn a sensitive answer |

**Design implications**: separate sensitive-answer table; exact-match filling only; unset answers stay blank (Q11); sensitive values never enter rules or shared files.

**Primary stories**: US-3.1, US-3.2, US-2.2, US-7.1, US-8.1

---

## P4 - Sam Rivera, the teammate who reviews shared rules

| Attribute | Detail |
| --- | --- |
| Background | A member of the project team who owns or reviews the "Rule learning and validation" stream |
| Technical comfort | High. Reads JSON, GitHub pull requests and CI logs |
| Devices | Any laptop with GitHub access |
| Goals | Approve only rules that replay correctly for all synthetic applicants and contain no personal values; respond within 48 hours (Q4); withdraw a bad rule quickly |
| Frustrations | Reviewing a rule without evidence that it works; rules that hide personal values in step text |
| What success looks like | A pull request shows a green schema check, a green personal-data scan and a green replay run, so the review is a short read of the steps |
| Trust boundary | Cannot approve a rule they authored; every withdrawal goes through a pull request |

**Design implications**: CI replay with Playwright on all fixture forms; index with checksums; removal from the index withdraws a rule; prefilled failure issues give reviewers the evidence they need (Q2).

**Primary stories**: US-6.2, US-6.3, US-6.4, US-10.1, US-10.2, US-10.3, US-10.4

---

## Persona to Epic Map

| Epic | P1 Maya | P2 Daniel | P3 Priya | P4 Sam |
| --- | --- | --- | --- | --- |
| E1 Profile Setup | Primary | Secondary | Secondary | - |
| E2 Fill an Application | Primary | Secondary | Secondary | - |
| E3 Sensitive Questions | - | - | Primary | - |
| E4 Open-Question Answers | - | Primary | - | - |
| E5 Teach a Fix | Primary | - | - | - |
| E6 Shared Fix Library | Primary (contributor) | - | - | Primary (reviewer) |
| E7 Review and Undo | Primary | Primary | Primary | - |
| E8 Data Control | Secondary | Primary | Primary | - |
| E9 Recovering from Failures | Primary | Primary | - | Secondary |
| E10 Demo Enablement | - | - | - | Primary |
