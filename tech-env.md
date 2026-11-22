# Job Application Assistant

## Purpose and deployment decision
The student installs the Chrome extension, signs in during setup, confirms their profile, and starts filling from an application page. Normal use requires no local server, terminal command, Python installation, or personal model API key. Personal records remain in extension storage. A team-operated cloud service handles model requests and a shared database distributes reviewed form repairs. This deployment proposal is aligned with README.md and vision.md. All three are input drafts for team review before Inception approval; the product and live cloud integration have not yet been implemented.

## Languages, packages, and frameworks
| Component | Proposed choice | Responsibility |
| --- | --- | --- |
| Chrome extension | TypeScript 5.x, Manifest V3, plain HTML/CSS, Vite | Side panel, profile settings, page inspection, execution, and background coordination. |
| Development tooling | Node.js 22 LTS, npm, TypeScript compiler | Build and test the extension. Commit package.json and package-lock.json. |
| Private local storage | IndexedDB in the extension origin, using the idb package | Persist confirmed profile records, scoped answers, personal repairs, run checkpoints and change history. |
| Hosted backend | Supabase Edge Functions, TypeScript on the managed Deno runtime | Authenticate requests, enforce quotas, call OpenAI, and manage rule-review operations. |
| Shared data and login | Supabase Postgres, Auth, and supabase-js v2 | Maintain accounts, private contributions, approved rules, versions, and revocations. |
| Resume parsing | Bundled pdfjs-dist and mammoth | Extract text from text-based PDF and DOCX locally. Parse DOCX as raw text. |
| Model | OpenAI Responses API over HTTPS, provisional gpt-4.1-mini | Extract structured profiles, interpret fields, match questions, and propose constrained repairs. |
| Validation and tests | Zod, Vitest, Playwright Test, Supabase CLI | Validate structured data, test pure logic and endpoints, exercise the extension, and test access policies. |

The team pins compatible package versions before Construction. Commit the npm and Deno dependency lockfiles. The included example uses TypeScript 5.8.3, Zod 3.24.2 and Vitest 3.1.3. Model selection remains provisional and follows evaluation on the project's extraction and repair cases. Students only install the built extension. Node, npm, Deno and CLI tools are developer tooling.

## Cloud provider and deployment model
The team deploys Edge Functions and database migrations to one Supabase project and configures HTTPS endpoints in the extension build. OpenAI inference is hosted separately. OPENAI_API_KEY and any privileged Supabase credential remain in function secrets. A public project key may be bundled in the extension, backed by authentication and row-level access policies. Email/password login with session refresh is the proposed initial account flow.

Known profile mappings and valid cached repairs execute locally immediately after a user starts filling. A new or ambiguous question can require a cloud request and display a waiting state. Automatic rule refresh runs independently of ordinary field entry. Free service tiers may cover a small demo but have quotas and availability limits. Model usage has its own cost. Team-wide and per-account limits control spending. Instant completion of network-dependent inference is not guaranteed.



## Where each category of data lives
| Data | Location and access | Lifecycle |
| --- | --- | --- |
| Confirmed personal profile | Extension-origin IndexedDB, associated with the signed-in applicant ID | School, address, GPA, work history and preferences stay on that browser installation. Students can edit and delete them. |
| Answers and application history | Same local store | Preserve application/company/general scope, before/after values and manual edits. These records are excluded from the shared rule catalog. |
| Raw resume and temporary text | Browser memory during extraction | Extract text locally, request structured extraction with notice, then discard the raw file and temporary text after confirmation or cancellation. |
| Model input | Necessary text passes through an authenticated Edge Function to OpenAI | Process transiently. Application code does not persist the request body in Postgres or logs. Provider handling still follows the selected API's data policies. |
| Submitted repair candidate | Private cloud submission owned by its contributor | Upload only after explicit contribution and sanitization. Contributor and authorized reviewers can access it. |
| Published repair and revocation | Cloud catalog with a local extension cache | Authenticated clients read reviewed versions and refresh revocations. Only authorized reviewers can publish or revoke. |
| Authentication and usage | Supabase Auth and restricted cloud usage tables | Store account identity, quotas, and non-content request metadata. Private applicant records are never made public through account creation. |

The product has separate storage for private applicant information and shared control knowledge. Personal values are inserted on the applicant's device when a shared rule runs. Local storage still sends the values required by an employer's form to that employer. Certain model requests also require personal context, which is disclosed during setup. Local storage does not imply that every processing step is offline.

## Local schema and recovery
Store profiles, education/work records, scoped answers, application contexts, runs, field changes, personal repairs and catalog cache entries in IndexedDB. Stable record IDs keep each degree or job connected to its dates. Approval and revision fields distinguish confirmed records from extraction candidates. A transaction records an operation's pending status before execution and its observed result after read-back. Operation IDs identify retries. Reinspect browser state after an interruption before repeating a write or a navigation.

Private records are keyed by applicant ID. Signing out clears authentication credentials and locks access through the extension UI. Switching accounts must not expose the previous account's records. Deleting the local profile also removes its answers, application contexts, run history, before/after values and pending demonstrations. Shared reviewed rules can remain cached. Browser-profile deletion or extension removal can lose local records. Cross-device profile synchronization and cloud backup are outside this initial proposal.

## Cloud access and retention
Use separate repair_submissions, published_rules, rule_revocations, reviewer_roles and usage tables. Row-level security restricts submissions to their owner and reviewers, exposes only approved public-schema rule fields to catalog readers, and prevents applicants from setting approval state or reviewer status. Reviewer roles are assigned through a trusted administrative path. Privileged service operations validate the caller and ownership explicitly. Usage and quota writes occur through a protected atomic server operation. Delete rejected or withdrawn submissions after 30 days. Contributors can request removal, while published sanitized rules follow the team's withdrawal policy established during Inception.



## Filling an application
The side panel reads the local confirmed profile and current application context. The content script inspects the permitted page and produces a snapshot of labels, headings, input types, listed options, existing values and repeated-section context. Existing mappings and cached verified rules are tried first. Unknown semantics can be sent to the model with the minimum necessary context. Model output references known profile fields and current snapshot control IDs. The local validator checks those references and current page state before execution.

The executor can set text, select a listed option, search an asynchronous selector, choose a matching candidate, add a supported repeated record, read values back, or follow an ordinary next step. A searchable school field is complete only after a matching offered option is selected and read back. Missing or ambiguous school options and record associations require student input. Inputs that reject the native setter and appropriate events become unresolved. The extension waits for observed state changes with timeouts.

Fill and correct authorizes replacing unambiguous pre-existing parser errors with confirmed values. Preserve before/after values for review and undo. Student edits after the run begins are protected. An execution guard distinguishes extension writes from user activity, while a final read immediately before writing detects concurrent edits. Undo restores a previous value only if the current value still matches the extension's result and the target remains identifiable. Reconcile repeated sections before adding records so rerunning does not create duplicates.

## Navigation and permissions
Request activeTab, scripting, storage and sidePanel plus host access to the configured Supabase endpoints. User invocation grants temporary access to the application tab. Reinject after supported same-origin reloads. Cross-origin navigation pauses for the student to invoke the extension on the destination. Unsupported cross-origin frames and closed shadow roots remain manual. These are explicit compatibility limits for the first release.

Persist checkpoints because Chrome may suspend the background service worker. On resume, identify the current tab and document, invalidate stale plans, inspect pending operations, and continue from verified state. Use one writer per application tab. An uncertain navigation result must be inspected before another click. Next and Continue actions are allowed only for identified step transitions. Agreements, signatures, accuracy certifications and final submission remain student-operated. Available page values and review summaries establish what the extension can verify.

## Answer scope and model requests
Saving a pasted answer records the exact question and answer, company, role, application URL and approval date locally. Default to current-application scope. Student confirmation is required to expand scope. Application answers require the same application ID. Company answers require confirmed company identity and approval to reuse on the target application. General answers still require semantic equivalence and compatible visible constraints. Missing details, ambiguous context or multiple plausible answers pause automatic reuse. Approved text is inserted unchanged.

A model match cannot broaden an answer's scope. Legal eligibility requires explicit student-provided answers for equivalent wording and country. New company-specific drafting and automatic answer shortening remain future options. Cloud requests use typed schemas and bounded tools. Set the provider timeout to 20 seconds and each background request attempt to 25 seconds, below Chrome's 30-second fetch-response boundary. At most two retries resume from persisted checkpoints within quota. Invalid output or exhausted retries pauses the affected task without discarding local progress.



## Learning, validation, and automatic distribution
The student chooses Teach this step and demonstrates a failed supported control. Capture only the selected control, its relevant structure, short action sequence and observed result. Replace personal values with profile references before requesting a candidate rule. The model proposes allowed operations and postconditions. The extension validates the proposal and checks its observed outcome. A personal rule becomes reusable only after that check succeeds.

A contributed candidate is uploaded only after explicit student consent and sanitization. Shared rules contain rule ID, version, compatible form signature, allowed operations, parameter references and assertions. Allowlisted literals describe control labels, option roles and structural features. Applicant answers, resume text, field values, account identifiers and private session URLs are excluded. Strict schemas and known-value checks support reviewer inspection. They cannot guarantee that arbitrary descriptive text contains no private information.

Before publication, a reviewer tests the candidate on two synthetic profiles and one incompatible or changed-control case. A successful result from one applicant is insufficient. Reviewers publish immutable versions. The extension checks catalog changes at sign-in and run start without blocking known direct profile filling. It validates schema and compatibility before activating a downloaded rule. Updates are installed transactionally, preserving the last valid version until the replacement validates.

Before each shared-rule execution, the client checks freshness against its last successful authenticated catalog/revocation check. The client caps this interval at 24 hours; server metadata cannot extend it. A known revocation disables the version immediately. If a cached version cannot be refreshed before its deadline, shared-rule execution pauses while direct known-field filling remains available. Failed local postconditions disable the rule for that form and offer a student-approved sanitized failure report. A reviewer can revoke or replace the version for all connected clients. Disconnected clients learn revocations on refresh, subject to the expiry boundary.

## Authentication, endpoint design, and secrets
The extension signs in once through Supabase Auth and stores refresh credentials only in trusted extension storage. Configure storage access so content scripts cannot read credentials. Only the background worker communicates with cloud endpoints. Validate message sender, tab and document identity and allow only fixed service destinations. Page-provided URLs cannot become arbitrary fetch requests.

Each Edge Function verifies the bearer token through Supabase Auth getUser or an equivalent validated JWT path and derives the user ID server-side. Browser-supplied user IDs and reviewer flags are untrusted. Apply ownership checks, exact allowed origins, schema validation, request limits, per-user quotas and a team budget. CORS supports browser isolation but does not replace authentication. Function secrets hold provider and privileged keys. Log request IDs, operation type, timing and token counts without raw resume or form payloads.

Proposed function endpoints include profile-extract, form-plan, question-match, repair-propose, repair-submit and repair-review. Profile extraction returns candidates to the local store. Catalog and revocation reads use authenticated read-only access policies. Write requests include idempotency keys and expected revisions. Conflicting revisions return 409. Model-dependent responses can return 429 for exhausted quota and 503 for temporary failure. Server-side actions and quota reservations use atomic database operations.



## Security basics and prohibited choices
Bundle all executable extension code. Webpage and document text are untrusted task data. Model output cannot authorize new operations, broaden scope or bypass review. Render text safely and use schema-validated JSON. Imported PDF/DOCX files have a proposed 10 MB limit and bounded parsing time and decompression size. Extract DOCX raw text, discard temporary files and avoid rendering imported markup. Browser account protection and OS disk encryption support local privacy. The MVP does not claim application-level encryption of IndexedDB.

| Prohibited choice | Reason | Use instead |
| --- | --- | --- |
| Local Python/FastAPI service required from students | Normal use must work after installation and setup. | Hosted Edge Functions and browser-side execution. |
| Selenium or Puppeteer as shipped runtime | Adds another browser process and duplicates control of the student's current tab. | Manifest V3 control adapters. Playwright is test tooling. |
| eval, new Function, remote scripts or executable repair packages | A learned or contributed rule could execute arbitrary code. | Constrained JSON operations handled by bundled code. |
| Unsafe YAML or executable serialization | Package parsing can construct unexpected objects or run code. | Size-limited JSON and strict Zod schemas. |
| Privileged database keys or OpenAI keys in the extension | Browser bundles expose privileged credentials. | Hosted function secrets and verified user sessions. |

## Testing and course acceptance
Use Vitest for field interpretation helpers, answer scope, operation validation, retry behavior and endpoint logic. Mock the model for deterministic tests. Use IndexedDB-compatible test storage for revision and recovery tests. Run separate Supabase integration tests with two ordinary accounts and one reviewer. Verify ownership isolation, rejection of forged approval flags, atomic quota limits, and publication/revocation permissions.

Use Playwright's bundled Chromium with a persistent context and unpacked extension. Test the three controlled application fixtures with three synthetic applicants, repeated records, asynchronous school search, document reloads, in-page step changes and protected manual edits. Simulate a field that loses its value, a background restart, cloud failure, cached-rule expiry, and a revoked rule. Manually smoke-test the extension in the team's installed Chrome. Actual employer-site coverage is recorded separately.

Retain at least 60 predefined field decisions, 90% correct eligible known values before human correction, 12 scope cases and two transferable repairs. Define eligible fields before each run and count unresolved eligible fields as failures. Designated tests require zero out-of-scope answer insertions, duplicate records on retry, protected edit overwrites or automated agreements/signatures/submissions. Two separate installations must demonstrate one reviewed repair arriving through the cloud and using different applicants' values. Record manual interventions and time to a reviewed form. Network-dependent steps are timed separately from direct local filling.

## Developer workflow and decisions
Use npm ci, npm run build, npm run typecheck, npm test and npx playwright test. Install test browsers with npx playwright install chromium. Supabase CLI applies reviewed migrations and deploys the functions using managed secrets. Developers may run local test services and fixtures, while students use deployed endpoints. Proposed directories are extension/, shared/, supabase/functions/, supabase/migrations/, tests/ and fixtures/. AI-DLC artifacts belong in aidlc-docs/.

Confirm service ownership, account flow, budget, compatible dependency pins, contribution deletion policy and fixture variants during Inception. The course permits a locally runnable product and also allows hosted services. The hosted design is a product choice to remove local-service startup. Review the aligned README, Vision and technical environment together before approving the input set.



## Example function and endpoint
This TypeScript example supplies a local scope gate and a hosted question-equivalence handler. A positive scope result only makes an answer a candidate. Question meaning, context and visible content limits must also pass before insertion. The server handler accepts question text, without the private answer or a requested scope override.

```typescript
import { z } from 'zod';

export function mayReuse(
  scope: 'application' | 'company' | 'general',
  sameApplication: boolean,
  sameCompany: boolean,
  approvedForTarget: boolean,
): boolean {
  if (scope === 'application') return sameApplication;
  if (scope === 'company') return sameCompany && approvedForTarget;
  return true;
}

const MatchInput = z.object({
  question: z.string().trim().min(1).max(2000),
  savedQuestion: z.string().trim().min(1).max(2000),
}).strict();
const MatchOutput = z.object({ equivalent: z.boolean() }).strict();

type Services = {
  authenticate: (request: Request) => Promise<string | null>;
  reserveQuota: (userId: string) => Promise<boolean>;
  classify: (input: z.infer<typeof MatchInput>) => Promise<unknown>;
};

export function createHandler(services: Services) {
  return async (request: Request): Promise<Response> => {
    if (request.method !== 'POST') return new Response(null, { status: 405 });
    const userId = await services.authenticate(request);
    if (!userId) return new Response(null, { status: 401 });
    let body: unknown;
    try { body = await request.json(); }
    catch { return new Response(null, { status: 400 }); }
    const input = MatchInput.safeParse(body);
    if (!input.success) return new Response(null, { status: 422 });
    if (!await services.reserveQuota(userId)) {
      return new Response(null, { status: 429 });
    }
    try {
      const result = MatchOutput.parse(await services.classify(input.data));
      return Response.json(result);
    } catch {
      return new Response(null, { status: 503 });
    }
  };
}
```

Production wiring supplies three services. authenticate validates the bearer token through Supabase Auth getUser. reserveQuota atomically reserves the authenticated account's allowance. classify calls OpenAI and returns an untrusted result for schema validation. The Edge Function wrapper handles allowed-origin preflight, bounded request size and unexpected infrastructure failures. The example tests these boundaries with fakes and does not establish live cloud integration.



## Example tests
Save the preceding code as answer.ts and this example as answer.test.ts. Run npx vitest run and npx tsc --noEmit in the example project. The full verification set also tests missing authentication, invalid schemas, quota exhaustion and malformed model output.

```typescript
import { expect, it, vi } from 'vitest';
import { createHandler, mayReuse } from './answer';

it('blocks a company answer at another company', () => {
  expect(mayReuse('company', false, false, true)).toBe(false);
});

it('returns a match without overriding application scope', async () => {
  const handler = createHandler({
    authenticate: vi.fn().mockResolvedValue('synthetic-user'),
    reserveQuota: vi.fn().mockResolvedValue(true),
    classify: vi.fn().mockResolvedValue({ equivalent: true }),
  });
  const request = new Request('https://example.test/question-match', {
    method: 'POST',
    body: JSON.stringify({
      question: 'Why this role?', savedQuestion: 'Why this role?',
    }),
    headers: { 'Content-Type': 'application/json' },
  });
  const response = await handler(request);
  expect(response.status).toBe(200);
  expect(await response.json()).toEqual({ equivalent: true });
  expect(mayReuse('application', false, false, false)).toBe(false);
});
```

## Reference documentation
1. Course input structure. https://github.com/awslabs/aidlc-workflows/tree/v1.0.1/docs/writing-inputs
2. Chrome activeTab access. https://developer.chrome.com/docs/extensions/develop/concepts/activeTab
3. Extension storage. https://developer.chrome.com/docs/extensions/develop/concepts/storage-and-cookies
4. Supabase Edge Functions. https://supabase.com/docs/guides/functions
5. Cloud access policies. https://supabase.com/docs/guides/database/postgres/row-level-security
6. Validating user sessions. https://supabase.com/docs/reference/javascript/auth-getuser
7. Extension browser tests. https://playwright.dev/docs/chrome-extensions
8. Database service pricing and free limits. https://supabase.com/pricing

[Download the Word version](docs/Technical_Environment_Job_Application_Assistant.docx)
