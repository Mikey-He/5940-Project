# Component Methods

**Version 2** (after the change request): adds tracking and ranking tools and methods; removes shared-library tools (`list_share_candidates`, `export_rule_candidate`, `draft_rule_issue`, `sync_rules`). Detailed business rules are defined per unit in Functional Design during Construction.

## Shared Types

```ts
type Result<T> = { ok: true; data: T } | { ok: false; code: ErrorCode; message: string };

type ErrorCode =
  | "INVALID_INPUT" | "NOT_FOUND" | "RUN_STATE_INVALID" | "FILE_MISSING" | "FILE_CHANGED"
  | "FILE_TYPE_REJECTED" | "FILE_TOO_LARGE" | "RULE_SCHEMA_INVALID" | "PERSONAL_DATA_FOUND"
  | "CONFIRMATION_REQUIRED" | "QUESTIONNAIRE_MISSING" | "WEIGHTS_INVALID" | "EVIDENCE_MISSING"
  | "STATUS_TRANSITION_INVALID" | "DUPLICATE_IGNORED" | "INTERNAL_ERROR";

type DocumentKind = "resume" | "cover_letter";
type Section = "contact" | "education" | "experience";
type ProfileRecord = { id: string; section: Section; fields: Record<string, string>; confirmed: boolean };
type Profile = Record<string, string>;          // flattened confirmed facts, e.g. "education.0.school"
type AnswerScope = { kind: "application"; applicationId: string } | { kind: "company"; company: string } | { kind: "general" };
type SensitiveCategory = "sponsorship" | "work_authorization" | "gender" | "race_ethnicity" | "veteran" | "disability";

type Field = { ref: string; label: string; section?: string; type: "text" | "select" | "combobox" | "date" | "radio" | "checkbox" | "file" | "textarea"; options?: string[]; wordLimit?: number; structure?: ControlStructure };
type ControlStructure = { role: string; tag: string; optionCount?: number; pattern?: string; parts?: number };
type PlannedAction =
  | { ref: string; action: "fill"; value: string; source: "profile" }
  | { ref: string; action: "rule"; ruleId: string; steps: ResolvedStep[] }
  | { ref: string; action: "sensitive"; value: string | null }
  | { ref: string; action: "upload"; kind: DocumentKind }
  | { ref: string; action: "answer"; question: string; wordLimit?: number }
  | { ref: string; action: "ask_claude"; reason: string }
  | { ref: string; action: "skip"; reason: string };
type StepType = "type" | "select_option" | "click" | "wait_for" | "read_back";
type RuleStep = { type: StepType; target: string; value?: string };
type Rule = { id: string; controlFingerprint: string; steps: RuleStep[]; enabled: boolean };
type ReadBack = { ref: string; value: string | null; decidedBy: "profile" | "rule" | "sensitive" | "claude" | "answer" | "upload" | "none" };
type RunState = "planned" | "filled" | "verified" | "closed";

// Tracking
type Status = "ranked" | "in_progress" | "submitted" | "interview_invited" | "interview_scheduled" | "offer" | "rejected";
type DisplayStatus = Status | "no_response";
type Application = { id: string; company: string; role: string; link?: string; appliedAt?: string; status: Status; updatedAt: string };
type StatusSource = { kind: "student" } | { kind: "message"; messageId: string } ;
type StatusProposal = { applicationId: string; newStatus: Status; evidence: string; source: StatusSource; confidence: "high" | "low" };

// Ranking
type Arrangement = "remote" | "hybrid" | "onsite";
type Questionnaire = { roles: string[]; industries: string[]; locations: string[]; arrangements: Arrangement[]; minPay: number | null;
  companySize?: string; startDate?: string; mustHaves: string[]; niceToHaves: string[];
  weights: { skills: number; interest: number; niceToHaves: number }; noResponseDays: number };
type PostingFacts = { location: string | null; arrangement: Arrangement | null; payMax: number | null };
type Judgement = { score: number; evidence: string[] };
type FitResult = { score: number; dealBreakers: string[]; unknowns: string[] };
```

## C4 - Tool Layer (MCP tools)

| Tool | Input | Output (`data`) | Calls |
| --- | --- | --- | --- |
| `register_document` | `{ path: string(≤1024), kind }` | `{ kind, fileName, size }` | C7 |
| `get_document` | `{ kind }` | `{ path }` or `FILE_MISSING` / `FILE_CHANGED` | C7 |
| `save_profile_records` | `{ records (≤100) }` | `{ added, duplicates, sections }` | C6 |
| `confirm_profile_section` | `{ section, edits? }` | `{ confirmed }` | C6 |
| `get_profile` | `{}` | `{ records }` (confirmed only) | C6 |
| `set_sensitive_answer` | `{ category, question(≤300), answer(≤200) }` | `{ category }` | C9 |
| `save_questionnaire` | `Questionnaire` | `{ saved: true }` or `WEIGHTS_INVALID` | C21 |
| `get_questionnaire` | `{}` | `Questionnaire` or `QUESTIONNAIRE_MISSING` | C21 |
| `plan_fill` | `{ runId?, pageUrl, pageFingerprint, fields (1..200) }` | `{ runId, pageIndex, plan }` | C5, C10 |
| `find_answers` | `{ question(≤500), company?, applicationId?, wordLimit? }` | `{ answers: { id, text, scope, old }[] }` | C8 |
| `check_answer` | `{ text(≤5000), wordLimit? }` | `{ words, withinLimit }` | C8 |
| `save_answer` | `{ text, scope, question }` | `{ id }` | C8 |
| `record_fill_result` | `{ runId, readBacks, originalValues }` | `{ review, disabledRules }` | C5, C11 |
| `undo_page` | `{ runId }` | `{ restore: { ref, value }[] }` | C5 |
| `close_run` | `{ runId, reason }` | `{ closed: true }` | C5 |
| `propose_rule` | `{ field, steps (1..20) }` | `{ ruleId }` or `RULE_SCHEMA_INVALID` / `PERSONAL_DATA_FOUND` | C11, C12 |
| `list_rules` | `{}` | `{ rules: { id, enabled, controlFingerprint }[] }` | C11 |
| `log_application` | `{ company(≤200), role(≤200), link?: url, jobId? }` | `{ applicationId, created: boolean }` | C22 |
| `confirm_submission` | `{ applicationId, date }` | `{ status: "submitted" }` | C22 |
| `update_status` | `StatusProposal` | `{ applied: boolean, pendingId?, undoId? }` or `DUPLICATE_IGNORED` | C22 |
| `list_pending_updates` | `{}` | `{ proposals: (StatusProposal & { id })[] }` | C22 |
| `resolve_pending_update` | `{ proposalId, decision: "approve" \| "reject" }` | `{ status }` | C22 |
| `undo_last_change` | `{ applicationId }` | `{ status }` | C22 |
| `list_applications` | `{ status?: DisplayStatus }` | `{ applications: (Application & { displayStatus, timeline })[] }` | C22, C21 |
| `export_tracker_csv` | `{ path, status? }` | `{ path, rows }` | C22 |
| `save_job` | `{ company, role, link?, postingText(≤20000) }` | `{ jobId, created }` | C23 |
| `score_job` | `{ jobId, facts: PostingFacts, skills, interest, niceToHaves: Judgement }` | `FitResult` or `EVIDENCE_MISSING` / `QUESTIONNAIRE_MISSING` | C23, C21, C22 |
| `compare_jobs` | `{ jobIds (1..20) }` | `{ jobs: { jobId, company, role, score, dealBreakers }[] }` (highest first) | C23 |
| `delete_my_data` | `{ confirm: true }` | `{ deleted }` | C14 |

29 tools in total. `tech-env.md` lists the core tools; this table adds the ones the stories need (for example `confirm_profile_section`, `check_answer`, `undo_page`, `close_run`, `list_rules`).

## C5 - Run Manager
```ts
startPage(runId: string | undefined, pageUrl: string, pageFingerprint: string, fieldCount: number): { runId: string; pageIndex: number };
transition(runId: string, to: RunState): Result<void>;
keepOriginals(runId: string, originals: { ref: string; value: string }[]): void;   // memory only
undoActions(runId: string): Result<{ ref: string; value: string }[]>;
recordOutcome(runId: string, plan: PlannedAction[], readBacks: ReadBack[]): ReviewList;
close(runId: string, reason: "next_page" | "done" | "abandoned"): void;
```

## C6 - Profile Service
```ts
saveExtracted(records: ProfileRecordInput[]): { added: number; duplicates: number };
confirmSection(section: Section, edits?: ProfileRecordEdit[]): number;
confirmedRecords(): ProfileRecord[];
flatten(records: ProfileRecord[]): Profile;
```

## C7 - Document Registry
```ts
register(path: string, kind: DocumentKind): Result<{ fileName: string; size: number }>;
resolve(kind: DocumentKind): Result<{ path: string }>;
hashFile(path: string): Promise<string>;
```

## C8 - Answer Service
```ts
find(question: string, ctx: { company?: string; applicationId?: string; wordLimit?: number }, now: Date): { id: string; text: string; scope: AnswerScope; old: boolean }[];
save(text: string, scope: AnswerScope, question: string, now: Date): string;
countWords(text: string): number;
isOld(savedAt: Date, now: Date): boolean;
```

## C9 - Sensitive Answer Guard
```ts
classify(field: Field, claudeMarkedSensitive?: boolean): SensitiveCategory | null;
setAnswer(category: SensitiveCategory, question: string, answer: string): void;
answerFor(field: Field, category: SensitiveCategory): string | null;
```

## C10 - Fill Planner
```ts
planFill(fields: Field[], profile: Profile, rules: Rule[], guard: SensitiveGuard): PlannedAction[];
normalizeLabel(label: string): string;
matchProfileKey(label: string): string | null;
```

## C11 - Rule Service
```ts
propose(field: Field, steps: RuleStepInput[], profile: Profile): Result<{ ruleId: string }>;
activeRules(): Rule[];
noteReadBack(ruleId: string, success: boolean): { disabled: boolean };
list(): { id: string; enabled: boolean; controlFingerprint: string }[];
```

## C12 - Rule Core
```ts
RuleSchema: z.ZodType<Rule>;
controlFingerprint(field: Field): string;
pageFingerprint(pageUrl: string, fields: Field[]): string;
resolveSteps(rule: Rule, profile: Profile): ResolvedStep[];
sanitize(steps: RuleStepInput[], profile: Profile): RuleStep[];
scanPersonalData(rule: Rule, profileValues: string[]): { ok: boolean; findings: string[] };
```

## C14 - Repository
```ts
open(dbPath: string): Db;
migrate(db: Db): void;
records, documents, answers, sensitive, questionnaire, rules, runs,
jobs, rankings, applications, statusHistory, pendingUpdates, processedMessages: typed parameterized query helpers;
deleteAll(db: Db): Record<string, number>;
```

## C15 - Logger
```ts
log(level: "info" | "warn" | "error", event: string, meta: { runId?: string; tool?: string; outcome?: string; durationMs?: number }): void;
```

## C21 - Questionnaire Service
```ts
validate(input: unknown): Result<Questionnaire>;        // weights sum to 1; defaults 0.5/0.3/0.2; noResponseDays default 21
save(q: Questionnaire): void;
get(): Questionnaire | null;
```

## C22 - Tracker Service
```ts
logApplication(input: { company: string; role: string; link?: string; jobId?: string }, now: Date): { applicationId: string; created: boolean };
confirmSubmission(applicationId: string, date: string): Result<void>;
canTransition(from: Status, to: Status, actor: "student" | "auto"): boolean;   // no automatic backwards moves; submitted only by student
propose(p: StatusProposal, matches: number, now: Date): Result<{ applied: boolean; pendingId?: string; undoId?: string }>;
resolvePending(proposalId: string, decision: "approve" | "reject"): Result<{ status: Status }>;
undoLast(applicationId: string): Result<{ status: Status }>;
displayStatus(app: Application, now: Date, noResponseDays: number): DisplayStatus;  // No response computed, never stored
list(filter: DisplayStatus | undefined, now: Date): (Application & { displayStatus: DisplayStatus; timeline: StatusChange[] })[];
toCsv(apps: Application[]): string;
```

## C23 - Fit Scorer
```ts
saveJob(input: { company: string; role: string; link?: string; postingText: string }): { jobId: string; created: boolean };
checkHardConstraints(q: Questionnaire, facts: PostingFacts): { dealBreakers: string[]; unknowns: string[] };
scoreFit(q: Questionnaire, facts: PostingFacts, skills: Judgement, interest: Judgement, nice: Judgement): FitResult;  // cap 35 on any deal-breaker
saveRanking(jobId: string, result: FitResult, inputs: object): void;
compare(jobIds: string[]): { jobId: string; score: number; dealBreakers: string[] }[];
```

## C17 - Replay Harness
```ts
executeRule(page: PlaywrightPage, rule: Rule, profile: Profile): Promise<{ ref: string; value: string }[]>;
replayAll(rules: Rule[], forms: string[], applicants: Profile[]): Promise<ReplayReport>;
```

## C18 - Demo Scorer
```ts
scoreFilling(runLogs: RunLog[], readBacks: ReadBack[][], answerKey: AnswerKey): { correct: number; total: number; unflaggedWrong: number };
compareCorrections(firstRun: RunLog[], laterRun: RunLog[], controls: string[]): { first: number; later: number };
scoreTracking(results: { messageId: string; applied: boolean; status?: Status }[], labels: MessageLabel[]): { accuracy: number; uncertainApplied: number };
scoreRanking(rankings: FitResult[], expected: PostingExpectations): { violationsFlagged: boolean; evidenceOnAll: boolean; top3: string[][] };
```
