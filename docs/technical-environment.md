# ApplyTrack AI · Technical environment

**INFO 5940 | AI-DLC v1.0.1 | Draft for team review**

ApplyTrack helps a student track applications to several roles, including roles at the same employer. A student can save a posting while applying or discover an application later from a confirmation email. Both paths feed the same local record. Each application shows what happened, what action is due, and the evidence for its current status.

The interaction experiment established the behavior to carry into Inception: capture a posting, confirm submission, inspect the source of an update, and link an event to one or more applications. Final visual design and usability evaluation remain open. The Chrome capture flow still requires an end-to-end test from the actual toolbar on the team's demo machine.

## 1. Runtime, dependencies, and deployment

| Component | Proposed environment | Responsibility |
| --- | --- | --- |
| Application | Python 3.12, FastAPI, Pydantic, Jinja2, browser JavaScript | Serve application histories and review controls. Validate requests and model output before applying changes. |
| Package management | Python `venv` and `pip` | Commit pinned dependencies for the demo during Construction and use the same environment for pytest. |
| Browser capture | Chrome Manifest V3 extension, plain JavaScript, `activeTab` and `scripting` | Open the local app or, on a user click, extract the current posting's URL, title and readable JD text for review. |
| Local database | SQLite via Python's `sqlite3` | Keep applications, events, event-to-application links, JD snapshots, source references, scan checkpoints and correction history together. |
| Gmail | Google OAuth and the Gmail API with `gmail.readonly` | Read messages in the authorized user's selected scan range. The backend owns synchronization. |
| Product model | OpenAI API, planned model `gpt-5.4-mini` | Classify relevant messages and extract structured fields. Select the final model by measured accuracy on our labeled messages, API access and cost. |
| Tests | pytest, FastAPI TestClient, temporary SQLite databases | Exercise extraction validation, linking, state changes, transaction retries and API behavior with isolated fixtures. |

**Deployment model.** The course demo runs on one laptop with one authorized test mailbox. The planned backend command is `uvicorn applytrack.main:app --host 127.0.0.1 --port 8765`. The user opens the dashboard in a browser. The extension sends user-confirmed captures to this same backend, so a later Gmail update finds the same records. Application data stays in local SQLite. Gmail access, direct posting retrieval and model inference use network services. OpenAI hosts inference. The team operates the web app locally. Codex is the development agent.

## 2. Capture and reconciliation

| Entry point | Evidence available | Expected behavior |
| --- | --- | --- |
| Student clicks the extension on a posting | Current page, original URL, selected or extracted JD, employer, title and requisition ID when present | Preview editable fields. Saving creates a **Started** application. An explicit “I applied” action records submission. Reopening a reliably identified posting offers its existing record. |
| Gmail finds a submission receipt | Source message ID, employer and any stated role, requisition ID, date or direct posting URL | Link a uniquely matched receipt to an existing capture and mark it **Applied**. Create a record for a clearly identified new application. Queue uncertain matches for review. |
| Email includes the original posting link | Public posting URL and readable page text | Retrieve the page and store the JD text, source URL and retrieval time as a snapshot. If retrieval fails or the email lacks the link, the user can paste the original JD. Keep the snapshot when the page expires. |

Use an employer plus a requisition ID or a corroborated original posting URL to identify an application. A title and a thread can provide supporting evidence. Reused threads, conflicting IDs, or multiple plausible matches require review. Preserve the actual submission date when supplied. Otherwise show when the receipt was received and leave submission time unknown. Preserve existing JD versions and user corrections when new evidence arrives.

**Scan lifecycle.** After Google authorization and confirmation of scan scope, run the initial scan automatically. Check periodically while the local app is running, offer a manual refresh, and catch up on reopening. Commit new events, links, and processed-message markers in a transaction. Advance the scan checkpoint only after all messages through that point have succeeded or been durably queued for review. A retry after a crash must give the same result. Use a unique key on mailbox and Gmail message ID for ingestion, and a separate event identity for an assessment or interview that several emails discuss.

## 3. Events, evidence, and user corrections

The LLM returns an event type, employer, role or requisition references, explicit scheduling details, and supporting text. Missing fields stay null. Python validates these fields and compares the evidence with stored applications. A model score alone cannot authorize a change.

| Situation | Decision and visible result |
| --- | --- |
| A single-role online assessment (OA) names a unique requisition | Attach the OA to that application. Retain the email reference and deadline if supplied. |
| One email explicitly says the **same OA** covers two identified roles | Store one assessment and two application links. Both histories refer to the same assessment and completion state. |
| One employer sends an OA without identifying a role, or lists two roles without explaining whether their OAs are shared | Keep the source in review. A student can drag it onto a candidate, inspect the source, select one or several roles and confirm. A click or keyboard selection performs the same action. Repositioning a node only changes its layout. Separate OAs retain separate event identities. |
| An employer invites the student to schedule or proposes a time | Show the invitation or proposal and the next action. Preserve any existing confirmed time until an explicit confirmation or reschedule arrives. |
| A confirmed meeting changes, is cancelled, or receives a reminder | Update the identified meeting, retain earlier times and sources, and count the interview once. Flag ambiguous times, time zones or rounds for review. |
| A user corrects or unlinks an event | Record the change and its previous links. A conflicting later scan requests review. Corrections and undo preserve an attributable history. |

An event-to-application link is unique on `(event_id, application_id)`. Store message provenance separately so a reminder can add evidence to an existing event. Keep unlinked events available for review. A lack of further email leaves an application awaiting response. The MVP presents interview dates inside ApplyTrack. Calendar writes, searching the web for a guessed JD, and automated job recommendations are outside this scope.

## 4. Security and data handling

- **Consent and least privilege.** Google presents the read-only Gmail grant. ApplyTrack separately explains the selected scan range and which message excerpts go to the hosted model. Use an authorized test account and synthetic recruiting messages for the course demo. Disconnect revokes access and stops scanning. Provide a separate local-data deletion action.
- **Extension and backend boundary.** Read the active posting only after the user clicks capture. Send the reviewed posting to the loopback backend using an explicitly paired extension and narrowly scoped local permission. Authenticate changes, validate the browser origin and protect dashboard forms against cross-site requests. Keep Gmail tokens and model credentials in the backend.
- **Minimize stored data.** Keep the message ID, necessary supporting text and extracted fields. Trim unrelated quoted conversations and signatures before model calls. Store OAuth tokens in the OS credential store and supply the API key through the local environment. Exclude credentials, real messages, JD captures and the local database from Git. Log processing outcomes without message bodies or credential-bearing links.
- **Treat external content as data.** Validate model responses against an allowed schema, escape displayed text and use parameterized SQL. For email-provided posting links, retrieve only public HTTPS destinations, validate each redirect, bound time and response size, and send no mailbox credentials or browser cookies.

## 5. Prohibited libraries and alternatives

These restrictions apply to the indicated uses in the course MVP.

| Library and prohibited use | Reason | Alternative |
| --- | --- | --- |
| Selenium or Playwright for signing in to job portals and retrieving private JDs | This would require handling a student's third-party session and adds a brittle dependency on portal login flows. | Capture the user-visible posting on click. For a public original URL use HTTPX and Beautiful Soup. Accept pasted JD text when unavailable. |
| Python `pickle` for importing mailbox, model or application records | Untrusted pickle data can execute code during deserialization. | Validate JSON with Pydantic and persist typed fields to SQLite. |

## 6. Representative endpoint, function, and test

**Endpoint.** A student inspects an ambiguous OA and confirms it belongs to two applications. The dashboard submits the complete selected set in one request. An empty set means “return to review.”

```http
PUT /api/events/oa-7/applications
Content-Type: application/json

{"application_ids": ["app-17", "app-23"], "expected_revision": 3}
```

The backend verifies the event and every selected application belong to the active account. In one transaction it replaces the link set, records the previous selection and updates the revision. Return `200` with the saved IDs and revision. A retry of the same selection returns the existing result without a second history entry. A stale revision with a different selection returns `409` and the current state for review. A failed ownership check changes nothing.

**Function.** This example handles OA emails *after* validation has resolved every role reference uniquely within the same employer and account. The caller sets `all_refs_resolved=False` for missing, conflicting or ambiguous references. The shared flag requires source wording that one assessment covers all named roles.

```python
def oa_link_targets(
    resolved_ids: tuple[str, ...], *, all_refs_resolved: bool, explicitly_shared: bool
) -> frozenset[str] | None:
    """Return links for one assessment, or None to request user review."""
    targets = frozenset(resolved_ids)
    if not all_refs_resolved or not targets:
        return None
    if len(targets) == 1 or explicitly_shared:
        return targets
    return None
```

**pytest example.** Two applications at the same company make the scope of the email consequential. Exercise that decision before any database write.

```python
import pytest


@pytest.mark.parametrize(("refs", "resolved", "shared", "expected"), [
    (("app-17",), True, False, {"app-17"}),
    (("app-17", "app-23"), True, True, {"app-17", "app-23"}),
    (("app-17", "app-23"), True, False, None),
    (("app-17",), False, True, None),
    ((), False, False, None),
])
def test_oa_links_require_evidence(refs, resolved, shared, expected):
    assert oa_link_targets(
        refs, all_refs_resolved=resolved, explicitly_shared=shared
    ) == expected
```

## 7. Verification during Construction

| Check | Evidence of success |
| --- | --- |
| Extension capture followed by Gmail confirmation | On the demo machine, capture a posting, inspect its JD and mark it started. Scan a matching receipt twice. Expect one application, one receipt event, the original JD snapshot and a single transition to Applied. Also test a receipt for an application that was never captured. |
| One shared OA versus two separate OAs | Replay synthetic messages for the same employer. A shared OA has one event ID and two links. Separate identified OAs have distinct IDs. A role-less OA stays in review. Confirming links twice leaves one correction entry. |
| Interview history | Replay invitation, proposed time, confirmation, reschedule, reminder and cancellation. Check the action due, the current appointment and the ordered history after each message. Repeat the scan and verify counts are unchanged. |
| Recovery and access boundaries | Simulate model timeout, interrupted scan, expired JD link, conflicting user correction and a request for another account's record. Verify recoverable work resumes, failed retrieval offers paste, and unrelated records stay intact. |
| Model selection | Label synthetic messages with expected event types, role scope and time fields. Compare classification errors, incorrect automatic links, missing-field handling and API cost on the same messages. Record the model ID and observed results for team review. |

The shared Inception session should reconcile these input decisions with the vision, especially Chrome capture and automatic scans. The team still needs to choose the first-scan window, polling interval, and final interaction design. The rules above give the team observable outcomes to use when reviewing user stories and acceptance criteria.
