# ApplyTrack AI technical environment

**Draft for team review · INFO 5940 · AI-DLC v1.0.1**

ApplyTrack helps a student answer “What happened with this application, and what do I need to do next?” A local dashboard shows one expandable timeline per application. An interview invitation, a proposed time, a confirmation, and a reschedule each carry different implications for the student. The system keeps the email evidence behind each change and lets the student correct the record.

## Runtime and deployment

| Component | Proposed choice | How we will use it |
| --- | --- | --- |
| Language and environment | Python 3.12, `venv`, `pip` | One virtual environment per checkout. During Construction, commit pinned dependencies for reproducible installs. |
| Local web app | FastAPI, Uvicorn, Jinja2 | Serve a dashboard, application timelines, and a review form from one Python process on the student's laptop. |
| Persistence | SQLite via Python's `sqlite3` | Store applications, events, message IDs, JD snapshots, and correction history across restarts. |
| Email access | Gmail API, Google OAuth, `gmail.readonly` | Read candidate messages from an authorized test account within the scan scope the user confirms. |
| Planned product LLM | OpenAI API, `gpt-5.4-mini`, hosted by OpenAI | Extract event type, employer, role, requisition ID, date, time zone, and supporting text into a Pydantic-validated schema. Missing values stay null. |
| Test framework | pytest, FastAPI TestClient, temporary SQLite databases | Run automated tests locally with synthetic emails and simulated Gmail, model, and job-page responses. |

The proposed demo runs on one laptop with one authorized account. During Construction, the planned entry point is `uvicorn applytrack.main:app --host 127.0.0.1`. The browser and database run locally. Gmail, the hosted LLM, and direct job-posting pages require internet access. Codex is the team's coding agent. The product calls the model through the backend using its own API credentials.

## How an email becomes a timeline event

1. **Scan.** After first-time authorization and scope selection, start an initial scan automatically. While the app runs, check periodically and offer “Scan now.” On reopening, resume from the last successful scan. Save progress after processing succeeds so an interrupted scan can retry.
2. **Extract and validate.** Send the relevant text of candidate emails to the model. Distinguish invitations, proposed times, confirmations, reschedules, cancellations, and reminders. Preserve the source time-zone text. If the sent date is the only date available, label it as the email date. Leave the application submission date unknown.
3. **Match.** Use a requisition ID or corroborating role and thread evidence to associate an update with a single application. Distinct roles at one employer get distinct timelines. An ambiguous match or a conflict with a user correction goes to review, where the user can select a record, create one, or edit fields.
4. **Record.** Commit the event, source reference, and processed-message marker in one database transaction. Use a unique key on account and Gmail message ID so a retry cannot apply a message twice. A separate reminder email can be linked to an existing interview. A reschedule updates that interview's displayed time while preserving its previous time and evidence. An application awaiting a reply stays pending.
5. **Display and correct.** Expand an application to see dated events and their sources. Show the currently scheduled interview separately from its history. Users can add missing applications and events, correct a field, or undo an automated change. Record each correction with its origin and time.

## Job descriptions

When the email contains a direct link to the original posting, fetch the public page and save its readable JD text, source URL, and retrieval time together. This preserves the version actually retrieved. If the posting is missing, expired, inaccessible, or cannot be identified on the page, offer a paste field. Label a pasted snapshot as user-provided. Keep a saved snapshot accessible after the link expires. The MVP uses only the supplied posting or pasted text for the JD. Interview schedules appear in the app.

## Security and data handling

- Use an authorized test mailbox containing synthetic recruiting messages for the course demo. Before scanning, show the mailbox scope and explain which email text is sent to the hosted provider. Strip signatures and irrelevant quoted history where possible while preserving evidence needed for a scheduling update.
- Keep OAuth tokens, API keys, local databases, and email contents outside Git. Store tokens in the OS credential store and read the model key from the local environment. Log message-processing outcomes without logging bodies, credentials, or token-bearing URLs. Provide account disconnect and local data deletion.
- Treat email and job-page content as input data. Validate model output against allowed fields and event types. Only application code can update SQLite. Escape text rendered in the dashboard.
- Fetch public HTTPS job pages with bounded response size and timeout. Restrict destinations to public addresses, recheck any redirect, and send no Gmail credentials, cookies, or authorization headers. On failure, preserve the application and offer manual JD entry.

## Prohibited libraries and alternatives for this MVP

| Prohibited use | Reason | Alternative |
| --- | --- | --- |
| Selenium or Playwright for retrieving JDs behind login | Browser automation would require managing third-party sessions and complicate the local demo. | Use HTTPX for public posting URLs, parse readable text with Beautiful Soup, and accept a pasted JD. |
| `pickle` for reading imported or model-produced records | Deserializing untrusted pickle data can execute code. | Validate JSON with Pydantic and persist the accepted fields in SQLite. |

## Representative endpoint, function, and test

These examples specify intended behavior for Construction. The review endpoint receives only the event ID and the application selected by the user. The backend resolves both against the current account before saving the decision.

```http
POST /api/review-events/evt-42/resolve
Content-Type: application/json

{"application_id": "app-17"}
```

On success, return `200` with the resolved event and application IDs. Repeating the same decision returns the existing result. A conflicting second decision returns `409` so the user can review the newer state.

The function below represents a decision after an interview has been identified and the extracted time has been normalized. Its return value helps the caller decide whether to change the displayed time. The caller retains the source message in either case.

```python
from datetime import datetime


def schedule_action(kind: str, new_time: datetime | None,
                    current_time: datetime | None) -> str:
    if kind not in {"confirmed", "rescheduled"}:
        return "keep_current_time"
    if new_time is None or new_time.utcoffset() is None:
        return "request_review"
    if new_time == current_time:
        return "keep_current_time"
    return "update_current_time"
```

```python
from datetime import datetime


def test_proposed_time_does_not_replace_confirmed_interview():
    tuesday = datetime.fromisoformat("2026-11-24T14:00:00-05:00")
    wednesday = datetime.fromisoformat("2026-11-25T14:00:00-05:00")
    assert schedule_action("proposed", wednesday, tuesday) == "keep_current_time"
    assert schedule_action("rescheduled", wednesday, tuesday) == "update_current_time"
```

During Construction, replay a labeled mailbox with two roles at one employer. Confirm a Tuesday interview for one role, reschedule it to Wednesday, send a reminder, then scan again. Expect one interview showing Wednesday, both schedule changes in its history, and no change to the other role. Also test expired posting links, missing time zones, and a user correction followed by a conflicting email. Evaluate the model separately on labeled messages and record event classification errors, incorrect extracted fields, and ambiguous matches incorrectly applied automatically.

## Decisions for the shared review

The proposed stack and model are working choices for the README and vision authors to review. Compare model cost and extraction accuracy on the same labeled emails before settling the Construction configuration. Align the vision's scan behavior with the proposed automatic initial scan, periodic checks while open, and catch-up on reopen. During requirements analysis, decide the initial scan window, polling interval, and what evidence is sufficient to apply an update automatically.
