# Component Methods

Signatures are TypeScript. Detailed business rules (matching tables, scan patterns, state transitions in full) are defined per unit in Functional Design during Construction.

## Shared Types

```ts
type Result<T> = { ok: true; data: T } | { ok: false; code: ErrorCode; message: string };

type ErrorCode =
  | "INVALID_INPUT" | "NOT_FOUND" | "RUN_STATE_INVALID" | "FILE_MISSING" | "FILE_CHANGED"
  | "FILE_TYPE_REJECTED" | "FILE_TOO_LARGE" | "RULE_SCHEMA_INVALID" | "PERSONAL_DATA_FOUND"
  | "NOT_SHAREABLE" | "SYNC_OFFLINE" | "CONFIRMATION_REQUIRED" | "INTERNAL_ERROR";

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
  | { ref: string; action: "sensitive"; value: string | null }        // null = leave blank and flag
  | { ref: string; action: "upload"; kind: DocumentKind }
  | { ref: string; action: "answer"; question: string; wordLimit?: number }
  | { ref: string; action: "ask_claude"; reason: string }
  | { ref: string; action: "skip"; reason: string };

type StepType = "type" | "select_option" | "click" | "wait_for" | "read_back";
type RuleStep = { type: StepType; target: string; value?: string };   // value: "{{profile.*}}" or a page option label
type Rule = { id: string; version: number; controlFingerprint: string; steps: RuleStep[]; origin: "personal" | "shared"; enabled: boolean };
type ReadBack = { ref: string; value: string | null; decidedBy: "profile" | "rule" | "sensitive" | "claude" | "answer" | "upload" | "none" };
type RunState = "planned" | "filled" | "verified" | "closed";
```

## C4 - Tool Layer (MCP tools)

| Tool | Input | Output (`data`) | Calls |
| --- | --- | --- | --- |
| `register_document` | `{ path: string(≤1024), kind: DocumentKind }` | `{ kind, fileName, size }` | C7 |
| `get_document` | `{ kind }` | `{ path }` or `FILE_MISSING` / `FILE_CHANGED` | C7 |
| `save_profile_records` | `{ records: ProfileRecordInput[] (≤100) }` | `{ added: number, duplicates: number, sections: Section[] }` | C6 |
| `confirm_profile_section` | `{ section, edits?: ProfileRecordEdit[] }` | `{ confirmed: number }` | C6 |
| `get_profile` | `{}` | `{ records: ProfileRecord[] }` (confirmed only) | C6 |
| `set_sensitive_answer` | `{ category, question: string(≤300), answer: string(≤200) }` | `{ category }` | C9 |
| `plan_fill` | `{ runId?: string, pageUrl, pageFingerprint, fields: Field[] (1..200) }` | `{ runId, pageIndex, plan: PlannedAction[] }` | C5, C10 |
| `find_answers` | `{ question: string(≤500), company?: string, applicationId?: string, wordLimit?: number }` | `{ answers: { id, text, scope, old: boolean }[] }` | C8 |
| `check_answer` | `{ text: string(≤5000), wordLimit?: number }` | `{ words: number, withinLimit: boolean }` | C8 |
| `save_answer` | `{ text, scope: AnswerScope, question }` | `{ id }` | C8 |
| `record_fill_result` | `{ runId, readBacks: ReadBack[], originalValues: { ref, value }[] }` | `{ review: ReviewList, disabledRules: string[] }` | C5, C11 |
| `undo_page` | `{ runId }` | `{ restore: { ref, value }[] }` | C5 |
| `close_run` | `{ runId, reason: "next_page" \| "done" \| "abandoned" }` | `{ closed: true }` | C5 |
| `propose_rule` | `{ field: Field, steps: RuleStepInput[] (1..20) }` | `{ ruleId }` or `RULE_SCHEMA_INVALID` | C11, C12 |
| `list_share_candidates` | `{}` | `{ rules: { ruleId, cleanReadBacks }[] }` | C11 |
| `export_rule_candidate` | `{ ruleId, consent: true }` | `{ filePath, json }` or `NOT_SHAREABLE` / `PERSONAL_DATA_FOUND` | C11, C12 |
| `draft_rule_issue` | `{ ruleId, runId }` | `{ title, body }` (no values) | C11 |
| `sync_rules` | `{}` | `{ added, removed, rejected, offline: boolean }` | C13 |
| `delete_my_data` | `{ confirm: true }` | `{ deleted: Record<string, number> }` | C14 |

`tech-env.md` listed 11 tools; Application Design adds eight (`confirm_profile_section`, `set_sensitive_answer`, `check_answer`, `undo_page`, `close_run`, `list_share_candidates`, `draft_rule_issue`, `delete_my_data`) to cover stories US-1.2, US-3.1, US-4.2, US-7.2, US-2.4, US-6.1, US-9.2 and US-8.1.

## C5 - Run Manager
```ts
startPage(runId: string | undefined, pageUrl: string, pageFingerprint: string, fieldCount: number): { runId: string; pageIndex: number };
transition(runId: string, to: RunState): Result<void>;              // rejects out-of-order moves
keepOriginals(runId: string, originals: { ref: string; value: string }[]): void;   // memory only
undoActions(runId: string): Result<{ ref: string; value: string }[]>;
recordOutcome(runId: string, plan: PlannedAction[], readBacks: ReadBack[]): ReviewList;
close(runId: string, reason: "next_page" | "done" | "abandoned"): void;   // discards originals
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
resolve(kind: DocumentKind): Result<{ path: string }>;     // FILE_MISSING, FILE_CHANGED
hashFile(path: string): Promise<string>;                    // SHA-256
```

## C8 - Answer Service
```ts
find(question: string, ctx: { company?: string; applicationId?: string; wordLimit?: number }, now: Date): { id: string; text: string; scope: AnswerScope; old: boolean }[];
save(text: string, scope: AnswerScope, question: string, now: Date): string;
countWords(text: string): number;
isOld(savedAt: Date, now: Date): boolean;                    // older than one season
```

## C9 - Sensitive Answer Guard
```ts
classify(field: Field, claudeMarkedSensitive?: boolean): SensitiveCategory | null;
setAnswer(category: SensitiveCategory, question: string, answer: string): void;
answerFor(field: Field, category: SensitiveCategory): string | null;   // exact match only
```

## C10 - Fill Planner
```ts
planFill(fields: Field[], profile: Profile, rules: Rule[], guard: SensitiveGuard): PlannedAction[];  // one action per field
normalizeLabel(label: string): string;
matchProfileKey(label: string): string | null;              // synonym table only
```

## C11 - Rule Service
```ts
propose(field: Field, steps: RuleStepInput[], profile: Profile): Result<{ ruleId: string }>;
activeRules(): Rule[];                                      // personal + cached shared, enabled only
noteReadBack(ruleId: string, success: boolean): { disabled: boolean };
shareCandidates(): { ruleId: string; cleanReadBacks: number }[];
exportCandidate(ruleId: string): Result<{ filePath: string; json: string }>;
issueBody(ruleId: string, runId: string): { title: string; body: string };
```

## C12 - Rule Core (shared)
```ts
RuleSchema: z.ZodType<Rule>;
controlFingerprint(field: Field): string;
pageFingerprint(pageUrl: string, fields: Field[]): string;
resolveSteps(rule: Rule, profile: Profile): ResolvedStep[];
sanitize(steps: RuleStepInput[], profile: Profile): RuleStep[];       // personal values -> {{profile.*}}
scanPersonalData(rule: Rule, extraValues?: string[]): { ok: boolean; findings: string[] };
```

## C13 - Rule Sync
```ts
sync(indexUrl: string, cache: RuleCache, fetchFn?: typeof fetch): Promise<{ added: number; removed: number; rejected: number; offline: boolean }>;
verifyChecksum(json: string, expectedSha256: string): boolean;
```

## C14 - Repository
```ts
open(dbPath: string): Db;
migrate(db: Db): void;
records: { insert, findDuplicates, confirmSection, listConfirmed };
documents: { upsert, get };
answers: { insert, search };
sensitive: { upsert, get };
rules: { insert, setEnabled, incrementCleanReadBacks, list };
ruleCache: { replaceAll, list };
runs: { insertPage, updateOutcome };
deleteAll(db: Db): Record<string, number>;
```

## C15 - Logger
```ts
log(level: "info" | "warn" | "error", event: string, meta: { runId?: string; tool?: string; outcome?: string; durationMs?: number }): void;
```

## C17 - Replay Harness
```ts
executeRule(page: PlaywrightPage, rule: Rule, profile: Profile): Promise<{ ref: string; value: string }[]>;
replayAll(rules: Rule[], forms: string[], applicants: Profile[]): Promise<ReplayReport>;
```

## C18 - Demo Scorer
```ts
score(runLogs: RunLog[], readBacks: ReadBack[][], answerKey: AnswerKey): { correct: number; total: number; flaggedWrong: number; unflaggedWrong: number };
compareCorrections(applicantA: RunLog[], applicantB: RunLog[], controls: string[]): { a: number; b: number };
```
