# Job Application Assistant
INFO 5940 · Fall 2026 · Vision · Team review draft · 22 November 2026

## Executive summary
Job Application Assistant is a Chrome extension for students applying for internships and early-career jobs. A student confirms personal information, education, employment, preferences, and reusable answers once, then starts filling from the employer's application page. The extension fills known information, corrects parsing errors, and checks the result across supported pages. When a control fails, a student can demonstrate the correction so the system can propose and validate a reusable repair. Reviewed repairs can help other applicants with matching forms while each applicant retains their own profile and answers.

## Problem and target users
Résumé uploads often produce incomplete or incorrect application forms. School selection, degree and date mapping, and repeated work records require manual repair. This work recurs across employers, even when the student's information is unchanged. Our initial problem evidence is the team's application experience, particularly repeated school-parsing errors. Broader usage and time savings remain hypotheses to evaluate during Construction.

Primary users are students with a résumé who are actively applying and want to spend less effort entering and checking information. Their relevant experience may include coursework, projects, internships, volunteering, and student leadership. They need control over which information is used and which answers apply to a specific employer. A secondary MVP role is a team reviewer who tests candidate form repairs and approves a version for the shared cloud catalog.

## Product value and the role of the LLM
The product combines a verified applicant profile with form execution, correction, and reuse of tested repairs. Existing browser assistants and application autofill products provide overlapping capabilities. The proposed value is reducing repeated intervention through a focused workflow and a maintained library of repairs. The project will demonstrate this behavior on a bounded set of forms and measure intervention counts.

The LLM extracts structured facts from résumé text, interprets fields in context, matches questions to approved answers, and proposes repair rules from demonstrations. A bounded agent can inspect a form, retrieve a profile reference, request an allowed action, and inspect the result. Program logic validates values, enforces answer scope, records changes, and controls navigation. Candidate repairs use profile references so the same behavior can operate on different applicants' values. Learning means accumulating verified rules and approved answers. Model training is outside the MVP.

## MVP delivery boundary
The baseline is a Chrome extension with automatically managed private browser storage and team-operated cloud services. Students install the extension, sign in and confirm their profile, then start filling directly from an application page. They do not start a local server, install developer tools or provide a personal model API key. Supabase Auth/Postgres holds accounts and reviewed shared rules; Edge Functions handle authenticated model requests. Known fields and valid cached repairs execute locally, while unfamiliar questions may require a cloud response. OpenAI GPT-4.1 mini is the provisional model, subject to team review and task-based evaluation. The demonstration uses three controlled English-language application fixtures and three synthetic applicants. At least one fixture is single-page and one has multiple steps. They cover different layouts and one deliberately changed control. Support for actual employer sites will be separately recorded after testing. Universal website support is a future goal.

## Private and shared data
Confirmed personal records, scoped answers, application contexts and correction history stay in the extension’s IndexedDB, separated by applicant account. The browser manages this storage automatically; normal use requires no database setup. Signing out clears credentials and locks access through the UI. Students can edit or delete their local profile and associated records. The MVP has no cloud profile backup or cross-device synchronization, so uninstalling the extension or deleting the browser profile can lose this data.

The shared cloud catalog stores reviewed control rules with profile references, not personal values. Pending submissions require explicit consent and remain visible only to their contributor and authorized reviewers. Cloud accounts and usage metadata are separate from local applicant profiles. Necessary résumé text or task context can pass through the hosted backend to the LLM; application code does not retain raw request content in cloud tables or logs. Provider handling follows the selected API’s policies. Cached direct filling does not wait for catalog refresh; model-dependent steps show progress and may pause on network failure.

## MVP features in scope
| Feature | Required behavior and purpose |
| --- | --- |
| Confirmed applicant profile | Import a text-based PDF or DOCX résumé, review extracted records, and add or edit missing information. Student confirmation establishes the source used for future filling. |
| Fill and correct | On a user click, fill known values and overwrite conflicting pre-existing values when the field and profile record match unambiguously. Show changes and support undo. |
| Supported controls and navigation | Handle text inputs, selects, radio buttons, searchable school selectors, and repeated education or work sections. Verify current-page results before following a supported Next or Continue action. |
| Scoped answer library | Store approved question-answer pairs with application, company, or general scope. Reuse only when the new question and context fit. |
| Learn a demonstrated repair | Capture a short, user-started demonstration of a failed supported control. Use the LLM to propose a reusable rule and verify its behavior. |
| Reviewed repair sharing | Automatically distribute approved, sanitized rule versions through a cloud catalog to two separate installations. Match applicability, check freshness and disable rules when validation fails. |
| Review and student control | Show filled values, corrections, and unresolved items with links to fields. Preserve manual edits, allow undo and resume, and leave agreements, signatures, and submission to the student. |

## Information confirmed before automatic use
| Record | Fields and distinctions |
| --- | --- |
| Identity and contact | Legal first and last name, optional middle and preferred names, email, phone, and country code. Legal and preferred names remain separate. |
| Address | Address lines, city, state or province, postal code, and country. Current address remains separate from preferred work location. |
| Each education record | Official school name, campus if relevant, degree, major, start month/year, graduation month/year, expected or completed status, and optional GPA with scale. Each date and GPA belongs to its own record. |
| Each work record | Employer, title, location, start month/year, end month/year or current employment, and approved description. Projects and volunteering retain their original category. |
| Links and preferences | Optional LinkedIn, portfolio and GitHub links, preferred work locations, work arrangement, relocation preference, and earliest start date. Preferences apply only to equivalent questions. |
| Approved answers | Original question, exact answer, source application URL, company and role context, chosen reuse scope, and last approval date. |

Missing facts remain unresolved. The student may add them when needed. GPA scales and degree equivalence are preserved as supplied. A relocation preference alone cannot establish willingness to accept a particular office arrangement. Work-authorization answers require explicit student input and approval for the relevant country and question. The extension never infers eligibility from a school, address, or résumé. Uncertain question matches remain unresolved.

## Filling, correction, and review behavior
The student opens an application, uploads the résumé through the employer's page, waits for its parsing to finish, and chooses Fill and correct. The extension reads labels, section headings, options, and existing values. It identifies which education or work record belongs to each repeated section before writing values. Ambiguous record matches become unresolved items. For a searchable school field, it types the school name, selects a matching offered option, and reads the selection back. Typing alone is insufficient. If a valid option is missing, the student handles the field.

The initial click authorizes correction of unambiguous pre-existing values using the confirmed profile, including a wrongly parsed school or graduation date. The review view records the previous and replacement values. Edits made by the student after the extension begins are preserved on subsequent runs. Undo restores changes where the page still supports it and identifies any field requiring manual restoration. Repeating a run must avoid duplicate education or work entries.

On supported multipage forms, the extension waits for page validation, follows an ordinary Next or Continue action, confirms that a new step loaded, and checks persistence through available page values or review summaries. An unresolved required field pauses progress and is highlighted. The student can supply it and resume. Controls that accept terms, sign declarations, or submit the application require student action even when their labels resemble continuation buttons. The review covers what can be read from the browser and cannot certify unseen employer-side storage.

## Saving and reusing open-question answers
Students can paste answers written by themselves or obtained from another tool. The extension asks whether to save the question and answer. It records the current company, role, and application URL and asks for missing context. The default scope is the current application. The LLM may suggest a broader scope, which the student confirms.

| Scope | Example | Reuse rule |
| --- | --- | --- |
| Current application | Why this opportunity, with an answer tied to this role's duties | Reuse only within that application. Other applications may use it as student-selected reference material. |
| Company | Motivation grounded in a company's product or mission | Offer for review at the same company. The student approves reuse because roles can differ. |
| General | An approved example of resolving a team disagreement | Fill when the question asks for the same content and the answer meets the current requirements. |

Matching considers the question's meaning, company and role context, and visible constraints such as word limits and requested details. An identical question at another company does not override scope. A company-specific answer without a company name still retains its saved scope. Mixed answers that combine personal experience and employer-specific motivation default to the current application. Missing requested details, incompatible limits, uncertain context, or multiple plausible answers require student input. The MVP preserves approved answer text. New company-specific drafting, automatic shortening, and automatic extraction of reusable fragments are future options.

## Learning and sharing a form repair
1. A control fails to accept or retain a value, or the student notices an incorrect mapping. The student starts Teach this step and demonstrates the local correction. Recording is limited to that control and ends when the student finishes.

2. The LLM receives the relevant control description and sanitized action sequence, then proposes a rule that references a profile field. For example, it can specify searching for the applicant's school, selecting the matching option, and checking the selected label. Personal values are substituted at execution time.

3. The executor checks that the rule uses supported operations and verifies its outcome. A verified personal rule may be reused on matching forms for that student. A failure remains visible and returns control to the student. A successful single demonstration is insufficient evidence for broader publication.

4. The student can contribute a sanitized candidate. A team reviewer checks the permitted actions and applicability and runs it against at least two synthetic profiles and one incompatible or changed-control case. Only rules that pass these checks enter the versioned cloud catalog. Personal answers, résumé text, field values, account identifiers, and session URLs are excluded from shared material. Pending contributions are private to their contributor and authorized reviewers. Authenticated installations automatically refresh approved versions and revocations at sign-in and run start; users do not export or import rule packages.

5. Another installation matches the rule to the relevant form structure and supported control behavior, then supplies its own applicant values. A shared recruitment-system brand alone is insufficient to establish compatibility. When applicability or read-back validation fails, that local rule is disabled for the affected form. The reviewer can revoke a version in the catalog. A known revocation disables it immediately. Before each shared-rule execution, the client enforces a maximum of 24 hours since its last successful authenticated catalog/revocation check. Expired rules pause if refresh fails, while ordinary known-field filling remains available. Disconnected clients receive revocations when they reconnect, subject to that freshness limit.

## User journeys
Maya imports her résumé and corrects a school and graduation date. She confirms two education records and two internships. On a single-page application, the employer parser has selected the wrong university. One click replaces the incorrect value, fills missing contact fields, and completes the matching work records. Maya reviews the changes and submits herself.

On a multipage form, Maya finds an unfamiliar school selector. She demonstrates the correct search and selection. The system proposes a candidate rule and checks that the chosen school remains after navigation. After review and testing, the rule enters the cloud catalog. Jordan’s separate installation refreshes the approved version automatically and completes an equivalent control using a different school. On a changed version of the control, validation prevents the incompatible rule from continuing.

Maya pastes an answer to Why are you interested in this opportunity and saves it for the current application. Another company's form uses the exact same question. The extension leaves it for her to answer. A saved general teamwork example is reused for an equivalent question when its content and length fit. Maya can change the inserted text and decide whether the revision should update her answer library.

## Success criteria for the December 16 demo
These are proposed acceptance targets for team review. Results must be recorded during Construction before any performance claim is made.

| Area | Demonstrable acceptance target |
| --- | --- |
| Profile and filling | Three synthetic applicants, each with two education and two work records. At least 60 predefined field decisions across three fixtures. At least 90% of eligible, known values are correctly filled before human correction. Define eligible fields from the supported control set and confirmed profile before each run. Count failed or unresolved eligible fields as failures. |
| Difficult controls | Include an incorrect school, a missing school option, swapped dates, a current job, an absent optional GPA, and a relocation question with different meaning. Missing facts and unsupported options remain unresolved in every designated test. |
| Navigation and persistence | Complete both single-page and multipage flows. Detect an injected value that disappears after moving pages. Pause at a missing required answer and resume after correction. Re-running adds zero duplicate records and preserves all designated manual edits. |
| Answer boundaries | At least 12 labeled question cases covering all three scopes, identical questions at different companies, changed word limits, and mixed answers. Zero out-of-scope insertions. All designated incompatible answers remain unresolved. |
| Learned repair transfer | Generate two repair candidates from recorded demonstrations. Each passes tests on two different synthetic profiles and one incompatible case before review. Automatically deliver approved rules through the cloud catalog to a second installation and use the second applicant's own values. |
| Rule failure and privacy | A changed-control case disables the applicable local rule. Published rules contain no synthetic personal values, answers, session identifiers, or private URLs. Test account isolation, reviewer-only publication, cached-rule expiry and revocation. Shared rules use constrained supported actions. |
| Review and control | Every extension overwrite appears in review with its previous value. Demonstrate undo and protection of subsequent manual edits. Zero automated agreements, signatures, or final submissions in the test suite. |
| Practical benefit | Record time to a reviewed, ready-to-submit form and manual interventions for manual and assisted runs. Counterbalance run order across teammates. Report results and failures without projecting a small team test to all students. |

## Constraints and delivery approach
Inception is due November 24, 2026. Construction and final delivery are due December 16. The extension runs on team laptops with private profiles in browser storage. Hosted Supabase services provide accounts, model-request mediation and reviewed rule distribution. The demo demonstrates reuse across two installations without requiring students to run local servers. The team owns cloud deployment, secrets and usage limits. All repository fixtures and recorded demo data are synthetic. API usage has a team-set spending limit. Source code, prompts, and tests must remain consistent with the scope approved through AI-DLC v1.0.1.

The proposed build order is a difficult school control and read-back verification, then confirmed profiles and repeated records, multipage execution and review, scoped answers, and finally demonstration-based repairs and cloud distribution. Establish account isolation and the cloud request boundary before processing personal data. The team prepares the fixture set before implementing repair generation so it can test transfer and failure. Controlled fixtures establish course-demo behavior. Employer-site compatibility requires separate verification and is reported by site and tested flow.

## Features explicitly out of scope
| Feature | Reason and possible future phase |
| --- | --- |
| Universal recruitment-site support, mobile browsers, scanned résumé OCR | The initial control and input set is bounded. Additional platforms and formats can follow the validated MVP. |
| Automatic final submission, agreements, signatures, and mass applications | Students retain these actions. They have no planned delivery phase in this project. |
| Login automation and CAPTCHA handling | Students complete these portions directly in the initial product. |
| New tailored answer generation, résumé or cover-letter rewriting | Students may paste content from their chosen tools. These could be evaluated after answer reuse works. |
| Gmail tracking, timelines, job recommendations, fit scoring, interview preparation | These features belong to the earlier concept. They are outside this product's current roadmap. |
| Public repair marketplace, automatic publication, cloud profile syncing | The MVP uses team review and automatic cloud distribution of approved rules. Public marketplace features and cross-device personal-profile sync remain future work. |
| Model fine-tuning and universal autonomous repair | The MVP learns through constrained, tested rules. Further learning methods require separate evaluation. |

## Risks and responses
Dynamic controls may appear filled while losing values. Read-back, page-transition checks, and explicit unresolved states make those failures observable. If a control cannot be supported in time, retain manual handling and report the coverage gap explicitly.

A demonstration may encode one person's values or work only on one layout. Profile references, multiple synthetic applicants, incompatible-control tests, and reviewer approval limit that generalization. Failed rules are disabled locally. The team tests catalog refresh, immediate handling of known revocations, and the client-enforced freshness limit for disconnected installations.

Model errors may attach a date to the wrong school or interpret a company-specific answer as general. Profile confirmation, scope enforcement, record-level checks, and preserved manual edits are required. LLM-proposed actions pass through a constrained executor. Webpage text is treated as task data, and page content cannot expand permitted actions.

Résumé text and selected task context may be sent to the hosted provider. Students are informed, can correct or delete local records, and choose whether to contribute a repair. Shared catalog entries contain reviewed control rules and synthetic verification material; private submissions have owner/reviewer-only access. Credentials, API keys, real résumés, and private application data stay out of GitHub and demo recordings.

## Full vision and open questions
Future work could expand the tested control library, browser and résumé-format support, optional private-profile synchronization and opt-in answer adaptation. Inception will confirm fixture coverage, ownership, budget, retention and acceptance targets, then divide the work into units. Application Design will refine the IndexedDB/Supabase schemas and execution plan proposed in tech-env.md.

[Download the Word version](docs/Vision_Job_Application_Assistant.docx)
