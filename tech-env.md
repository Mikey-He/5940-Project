# Job Application Assistant technical environment

## Purpose and deployment decision

Job Application Assistant is a Claude Code plugin with three features described in `vision.md`: apply (fill application forms from a confirmed profile), track (keep one record of every application and its status) and rank (a 0-100 fit score for a job posting). Claude is the LLM and the agent. The plugin's skills describe each workflow, and Claude in Chrome reads and operates the employer's page in the student's browser. A local MCP server that we write in TypeScript stores the student's data, plans each fill, checks every rule, applies status rules and computes fit scores. The guiding rule is simple. Claude interprets language. Code makes every decision that must come out the same way each time, and every such decision is unit tested.

## Language and version, with framework and package manager

| Component | Choice | Responsibility |
| --- | --- | --- |
| Host and LLM | Claude Code with the student's own Claude plan. We develop and test with Claude Sonnet 5.5. | Reads pages and résumés. Chooses tools and talks to the student. |
| Plugin format | Claude Code plugin with `.claude-plugin/plugin.json`, `skills/<name>/SKILL.md` and `.mcp.json` | Packages the skills and the MCP server |
| Browser | Claude in Chrome extension, started with `claude --chrome` | Reads the page and fills fields. Also uploads the résumé file |
| Email (optional, stretch goal) | Claude's Gmail connector, read only | Lets Claude find employer replies when the student asks |
| MCP server language | TypeScript 5.x with `@types/node`, on Node.js 22.13 or later, ES modules | Our own code |
| MCP framework | `@modelcontextprotocol/sdk`, stdio transport | Exposes typed tools to Claude |
| Validation | Zod 3 | Schemas for tool inputs and stored records. Rules too |
| Local database | Built-in `node:sqlite` module | Profile, answers, questionnaire, personal rules, run log, jobs, rankings, applications and status history |
| Bundler | esbuild, one file at `server/dist/index.js`, committed | Teammates install the plugin without running `npm install` |
| Package manager | npm with a committed `package-lock.json`, installed with `npm ci` | Reproducible installs |

## Cloud provider and deployment model

The product runs locally. There is no cloud deployment and no server of our own. Each student installs the plugin in Claude Code, and Claude Code starts our MCP server on the laptop over stdio when a session begins. Developers load the working copy with `claude --plugin-dir ./plugins/job-application-assistant --chrome`.

The only hosted service is Claude itself, which runs on Anthropic's service under the student's plan. If the student connects it, Claude's read-only Gmail connector is used to look for employer replies when the student asks. The MCP server opens no network ports and makes no network requests.

## Plugin layout

```text
plugins/job-application-assistant/
  .claude-plugin/plugin.json      name, version, description, author
  .mcp.json                       starts server/dist/index.js with node
  skills/
    set-up-profile/SKILL.md       résumé to confirmed records
    fill-application/SKILL.md     plan, batch fill, read back, review, log the application
    teach-a-fix/SKILL.md          demonstration to personal rule
    track-applications/SKILL.md   status updates, tracker table, CSV export
    rank-this-job/SKILL.md        extract posting facts, score 0-100, compare saved jobs
  server/
    src/  index.ts db.ts profile.ts answers.ts questionnaire.ts plan.ts rules.ts sanitize.ts tracker.ts score.ts csv.ts
    test/ *.test.ts
    dist/index.js                 bundled output, committed
.github/workflows/ci.yml          build, unit tests, audit, fixture replay
fixtures/forms/                   three static fixture forms
fixtures/applicants/              three synthetic applicants
fixtures/messages/                at least 20 labeled synthetic employer messages
fixtures/postings/                at least 10 synthetic postings and two questionnaires
```

There is no `bin/` folder, because Cowork refuses to install a plugin that has one. No name starts with `claude-`.

The `.mcp.json` file looks like this.

```json
{
  "mcpServers": {
    "job-apply": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/server/dist/index.js"],
      "env": {
        "JOBAPP_DB": "${CLAUDE_PLUGIN_DATA}/jobapp.db"
      }
    }
  }
}
```

## Where each kind of data lives

| Data | Location | Notes |
| --- | --- | --- |
| Résumé and cover letter files | Stay where the student keeps them | The database stores each file's path along with its size and SHA-256 hash. Nothing is copied |
| Confirmed profile records | SQLite in `${CLAUDE_PLUGIN_DATA}` | Kept across plugin updates and never committed. Uninstalling deletes it unless the student keeps the data. |
| Open-question answers with scope | Same database | Scope is either one application or one company. General answers apply everywhere |
| Sensitive fixed answers | Same database, separate table | Filled only by exact question match. Never sent to a rule and never drafted |
| Personal rules and fill history | Same database | History stores field labels and outcomes, not values |
| Preference questionnaire | Same database | Roles, locations, arrangements, minimum pay, weights, No response cutoff |
| Jobs and rankings | Same database | Posting facts, the three judgements with evidence quotes, the final score, deal-breakers and unknowns |
| Applications and status history | Same database | Company, role, link, dates, current status, every status change with its source, pending proposals and undo entries |
| Exported tracker CSV | A file the student chooses | Never committed |
| Model input | Sent to Claude by Claude Code | Page text plus the profile fields that page needs |

## Giving the résumé and cover letter

The student drags the PDF into the Claude Code prompt or types its path. Both give Claude the file's location, and Claude reads the file straight from disk. Claude then calls `register_document`, so later sessions find the file without asking again. Drag and drop behaves differently across terminals, so the setup skill always mentions typing the path as the fallback.

## MCP tools (the endpoints Claude calls)

| Tool | Input | What code does |
| --- | --- | --- |
| `get_profile` | none | Returns confirmed records only |
| `register_document` | File path and kind (résumé or cover letter) | Accepts only PDF or DOCX up to 10 MB. Stores the path with its size and hash |
| `get_document` | Kind | Confirms the file still exists and is unchanged, then returns its path. Otherwise asks Claude to request the file again |
| `save_profile_records` | Records extracted by Claude | Validates and deduplicates them, then stores them as unconfirmed until the student confirms |
| `plan_fill` | Page URL and fingerprint. A list of fields, each with its label and type plus options and an element reference | Matches fields to profile facts and saved rules. Returns a plan plus the fields left for Claude |
| `record_fill_result` | Run ID and read-back values | Compares them with the plan and flags mismatches. Disables a failing rule locally |
| `find_answers` | Question text and word limit | Returns scoped saved answers that fit the limit |
| `save_answer` | Answer text and scope | Stores an approved answer |
| `propose_rule` | Steps from a demonstration | Validates the steps and replaces personal values with profile references. Saves the result as a personal rule |
| `save_questionnaire`, `get_questionnaire` | Preferences and weights | Validates values; weights must add up to 100% |
| `log_application` | Company, role, link | Creates or reuses the tracker entry (no duplicates by link) with status In progress |
| `confirm_submission` | Application ID, date | Moves the application to Submitted only after the student confirms |
| `update_status` | Application ID, new status, evidence quote, source, confidence | Applies a clear update with an undo entry; stores an uncertain one as a pending proposal |
| `list_pending_updates`, `resolve_pending_update` | Proposal ID, approve or reject | Lets the student confirm or reject uncertain changes |
| `undo_last_change` | Application ID | Reverts the latest status change or tracker edit |
| `list_applications`, `export_tracker_csv` | Optional status filter | Tracker sorted by date applied with timelines; computes No response; writes a CSV |
| `save_job` | Company, role, link or posting text | Stores a posting for ranking; no duplicates by link |
| `score_job` | Job ID, posting facts, three judgements with evidence | Checks hard constraints, computes the 0-100 score, saves it and sets the tracker status to Ranked |
| `compare_jobs` | Job IDs | Saved scores side by side, highest first |

## Filling an application

1. The fill skill asks Claude to read the page structure once and list every field with its label and type, plus any options.
2. Claude calls `plan_fill`. Code fills what it can match exactly. It attaches a saved rule to any control the rule's fingerprint matches.
3. Claude decides only the fields code could not place, using the label and its section. For a file field, Claude calls `get_document` and uploads that file through Claude in Chrome.
4. Claude executes the plan in one batch with the browser's form tools and then reads every value back.
5. Claude calls `record_fill_result` and shows the review list. The plugin logs the application in the tracker with `log_application`. The student checks the page, submits it personally, and then confirms the submission so the tracker shows Submitted.

Claude in Chrome pauses at login pages and CAPTCHAs for the student. The skill never clicks a submit button.

## How a personal rule is learned

A rule is a small JSON document with a target fingerprint and a list of steps. Values are profile references such as `{{profile.education[0].school}}`. A step must be one of these five types.

- `type`
- `select_option`
- `click`
- `wait_for`
- `read_back`

Validation has three layers.

1. **Schema.** Zod rejects unknown step types and literal values. The personal-data scan rejects anything that looks like personal data, including email addresses, phone numbers and any value in the student's profile.
2. **Replay in tests.** Playwright replays rules on the fixture forms for all three synthetic applicants in CI.
3. **Runtime.** Each filled value is read back. A mismatch disables that rule and flags the field. A fingerprint mismatch skips the rule.

Rules stay on the student's laptop. Sharing fixes across students is out of scope for the MVP.

## Tracking and status rules

Application statuses form a fixed ordered set: Ranked, In progress, Submitted, Interview invited, Interview scheduled, then Offer or Rejected. No response is computed, never stored.

- Only the student can move an application to Submitted (`confirm_submission`).
- A Submitted application with no later update after the cutoff (default 21 days, set in the questionnaire) is shown as No response.
- A status never moves backwards automatically.
- Claude classifies an employer message the student pastes or (stretch goal) finds through the read-only Gmail connector. A high-confidence update that matches exactly one application and does not conflict with an earlier student edit applies automatically with an undo entry. Anything else becomes a pending proposal.
- Recording the same message twice changes nothing, and logging the same link again updates the existing entry.

## Fit scoring

- Claude extracts the posting's location, work arrangement and top of the pay range, and judges skills fit, interest fit and nice-to-have fit from 0 to 100, each with quotes from the posting as evidence.
- Code checks the hard constraints from the questionnaire: location, work arrangement and minimum pay. A violated constraint is a deal-breaker and caps the score at 35. A fact the posting does not state is listed as unknown, not treated as a violation.
- Code combines the three judgements with the questionnaire weights (default skills 50%, interest 30%, nice-to-haves 20%).

## Security basics and prohibited choices

- Page text and résumé text are data, not instructions, and so are job descriptions, postings and employer messages. Skills tell Claude to ignore instructions found in them. Text on a page can never add a tool call that the student did not ask for.
- Every MCP tool validates its input with Zod and refuses anything that fails.
- The plugin never submits an application or enters a password. It never creates an account or answers a sensitive question from a rule. It never marks an application Submitted without the student's confirmation.
- No secrets exist in this product. No API keys are stored, because Claude Code handles the model connection under the student's login.
- Logs record tool names and outcomes with timings, never profile values.
- Only synthetic data is committed. `.gitignore` excludes `node_modules/`, `*.db`, `.env` and exported tracker CSV files.
- Gmail access, when used, is read only through Claude's connector. The plugin never sends, labels or deletes mail.

| Prohibited | Reason | Use instead |
| --- | --- | --- |
| `better-sqlite3` or other native database modules | A native binary breaks the single bundled file on other operating systems | Built-in `node:sqlite` |
| Puppeteer or Selenium inside the plugin | Browser control must happen through Claude in Chrome, in the student's visible browser with its permission prompts | Claude in Chrome tools. Playwright is allowed only in tests |
| OpenAI or Anthropic SDKs inside the plugin | Claude Code already provides the model, and an SDK would need an API key | Skills and tool calls in the host Claude session |
| `axios` or `node-fetch` | Node 22 has `fetch` built in, so the extra dependency is unnecessary | Global `fetch` |
| Unofficial ATS or LinkedIn scraping libraries | They break site terms, and our MVP does not need them | Fixture forms and the student's open page |
| `eval` or `new Function` on rule content | A rule would become executable code | Interpret the fixed step types only |
| Gmail write access or a custom mail integration | We never send, label or delete mail | Claude's read-only Gmail connector |

## Test framework

| Layer | Tool | What it covers |
| --- | --- | --- |
| Unit | Vitest | Field matching and the planner. Rule schema and the personal-data scan. Answer scope. Status rules, No response, pending updates and undo. Hard constraints and fit scoring. CSV export |
| Rule replay | Playwright Test with the three fixture forms | Every taught rule fills every synthetic applicant correctly |
| Plugin checks | `claude plugin validate --strict` | Manifest and MCP entry, including paths |
| Scenario check | Manual run with Claude Code and Chrome, recorded | The seven success criteria in `vision.md`, including 20 labeled messages and 10 ranked postings |

Run `npm ci` and `npm run build`, then `npm test` and `npx playwright test`.

## Example function (plan a fill)

```ts
// server/src/plan.ts. Code decides every field it can, and Claude handles only the rest
import { z } from "zod";

export const Field = z.object({
  ref: z.string(),                        // element reference from the page read
  label: z.string().max(300),
  type: z.enum(["text", "select", "combobox", "date", "radio", "checkbox", "file", "textarea"]),
  options: z.array(z.string()).optional(),
});
export type Field = z.infer<typeof Field>;

export type Profile = Record<string, string>;   // keys such as "contact.email", "education.0.school"
export type Rule = { id: string; fingerprint: string; profileKey: string; steps: unknown[] };
export type Planned =
  | { ref: string; action: "fill"; value: string; source: "profile" }
  | { ref: string; action: "rule"; ruleId: string; value: string }
  | { ref: string; action: "ask_claude"; reason: string };

const SYNONYMS: Record<string, string[]> = {
  "contact.email": ["email", "e-mail", "email address"],
  "contact.phone": ["phone", "mobile", "phone number"],
  "education.0.school": ["school", "university", "institution", "school name"],
};

const normalize = (s: string) => s.toLowerCase().replace(/[^a-z ]/g, "").trim();

export function planFill(fields: Field[], profile: Profile, rules: Rule[], pageFingerprint: string): Planned[] {
  return fields.map((raw) => {
    const field = Field.parse(raw);
    const label = normalize(field.label);
    const key = Object.keys(SYNONYMS).find((k) => SYNONYMS[k].includes(label));
    if (!key || profile[key] === undefined) {
      return { ref: field.ref, action: "ask_claude", reason: "no exact profile match" };
    }
    const rule = rules.find((r) => r.fingerprint === pageFingerprint && r.profileKey === key);
    if (rule) return { ref: field.ref, action: "rule", ruleId: rule.id, value: profile[key] };
    if (field.type === "combobox") {
      return { ref: field.ref, action: "ask_claude", reason: "searchable control without a saved rule" };
    }
    return { ref: field.ref, action: "fill", value: profile[key], source: "profile" };
  });
}
```

## Example endpoint (the `plan_fill` MCP tool)

```ts
// server/src/index.ts
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";
import { Field, planFill } from "./plan.js";
import { openDb, confirmedProfile, cachedRules, startRun } from "./db.js";

const db = openDb(process.env.JOBAPP_DB!);
const server = new McpServer({ name: "job-apply", version: "0.1.0" });

server.registerTool(
  "plan_fill",
  {
    description:
      "Plan how to fill the fields on the current application page. Returns one entry per field. " +
      "Fill 'fill' and 'rule' entries exactly as given. Decide only 'ask_claude' entries yourself.",
    inputSchema: {
      pageUrl: z.string().url(),
      pageFingerprint: z.string().min(8).max(128),
      fields: z.array(Field).min(1).max(200),
    },
  },
  async ({ pageUrl, pageFingerprint, fields }) => {
    const plan = planFill(fields, confirmedProfile(db), cachedRules(db), pageFingerprint);
    const runId = startRun(db, pageUrl, plan.length);   // stores labels and actions, never values
    return {
      content: [{ type: "text", text: JSON.stringify({ runId, plan }) }],
      structuredContent: { runId, plan },
    };
  },
);

await server.connect(new StdioServerTransport());
```

## Example function (fit score)

```ts
// server/src/score.ts. Hard constraints and the final score are computed in code
export type Arrangement = "remote" | "hybrid" | "onsite";
export type Questionnaire = {
  locations: string[]; arrangements: Arrangement[]; minPay: number | null;
  weights: { skills: number; interest: number; niceToHaves: number };   // sum to 1
};
export type PostingFacts = { location: string | null; arrangement: Arrangement | null; payMax: number | null };
export type Judgement = { score: number; evidence: string[] };          // 0-100 with quotes from the posting
export const DEAL_BREAKER_CAP = 35;

export function scoreFit(q: Questionnaire, f: PostingFacts, skills: Judgement, interest: Judgement, nice: Judgement) {
  const dealBreakers: string[] = [];
  const unknowns: string[] = [];
  if (f.arrangement === null) unknowns.push("arrangement");
  else if (!q.arrangements.includes(f.arrangement)) dealBreakers.push("arrangement: " + f.arrangement);
  if (f.arrangement !== "remote") {
    if (f.location === null) unknowns.push("location");
    else if (!q.locations.some((l) => l.toLowerCase() === f.location!.toLowerCase())) dealBreakers.push("location: " + f.location);
  }
  if (q.minPay !== null) {
    if (f.payMax === null) unknowns.push("pay");
    else if (f.payMax < q.minPay) dealBreakers.push("pay below minimum");
  }
  if (skills.evidence.length === 0 || interest.evidence.length === 0) throw new Error("Judgements need evidence from the posting");
  const w = q.weights;
  const raw = w.skills * skills.score + w.interest * interest.score + w.niceToHaves * nice.score;
  return { score: Math.round(dealBreakers.length ? Math.min(raw, DEAL_BREAKER_CAP) : raw), dealBreakers, unknowns };
}
```

## Example test

```ts
// server/test/plan.test.ts
import { describe, it, expect } from "vitest";
import { planFill } from "../src/plan";

const profile = { "contact.email": "maya.test@example.edu", "education.0.school": "Example State University" };
const school = { ref: "f2", label: "School name", type: "combobox" as const };

describe("planFill", () => {
  it("fills an exact text match from the confirmed profile", () => {
    const [p] = planFill([{ ref: "f1", label: "Email Address", type: "text" }], profile, [], "fp-greenhouse-1");
    expect(p).toEqual({ ref: "f1", action: "fill", value: "maya.test@example.edu", source: "profile" });
  });

  it("hands a searchable control to Claude when no saved rule exists", () => {
    const [p] = planFill([school], profile, [], "fp-workday-1");
    expect(p.action).toBe("ask_claude");
  });

  it("uses a saved rule only when the page fingerprint matches", () => {
    const rule = { id: "r-school", fingerprint: "fp-workday-1", profileKey: "education.0.school", steps: [] };
    expect(planFill([school], profile, [rule], "fp-workday-1")[0].action).toBe("rule");
    expect(planFill([school], profile, [rule], "fp-workday-2")[0].action).toBe("ask_claude");
  });

  it("never guesses a field the profile does not cover", () => {
    const [p] = planFill([{ ref: "f9", label: "Favorite color", type: "text" }], profile, [], "fp");
    expect(p).toMatchObject({ action: "ask_claude" });
  });
});

// server/test/score.test.ts
import { scoreFit, type Questionnaire } from "../src/score";
const q: Questionnaire = { locations: ["New York, NY"], arrangements: ["remote", "hybrid"], minPay: 70000,
  weights: { skills: 0.5, interest: 0.3, niceToHaves: 0.2 } };
const j = (score: number) => ({ score, evidence: ['"SQL and Tableau required"'] });

describe("scoreFit", () => {
  it("caps an on-site job in another state at 35", () => {
    const r = scoreFit(q, { location: "Austin, TX", arrangement: "onsite", payMax: 90000 }, j(90), j(80), j(50));
    expect(r.score).toBe(35);
  });
  it("lists missing pay as unknown, not a deal-breaker", () => {
    const r = scoreFit(q, { location: null, arrangement: "remote", payMax: null }, j(70), j(70), j(70));
    expect(r).toMatchObject({ score: 70, unknowns: ["pay"], dealBreakers: [] });
  });
});
```

## Reference documentation

- [Claude Code plugin manifest reference](https://code.claude.com/docs/en/plugins-reference)
- [Claude Code with Chrome](https://code.claude.com/docs/en/chrome)
- [Plugins in Cowork](https://support.claude.com/en/articles/13837440-use-plugins-in-cowork)
- [Model Context Protocol TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk)
- [AI-DLC v1.0.1](https://github.com/awslabs/aidlc-workflows/releases/tag/v1.0.1)
