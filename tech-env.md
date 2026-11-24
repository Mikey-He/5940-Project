# Job Application Assistant technical environment

## Purpose and deployment decision

Job Application Assistant is a Claude Code plugin. Claude is the LLM and the agent. The plugin's skills describe each workflow, and Claude in Chrome reads and operates the employer's page in the student's browser. A local MCP server that we write in TypeScript stores the student's data and plans each fill. It also checks every rule. The guiding rule is simple. Claude interprets language. Code makes every decision that must come out the same way each time, and every such decision is unit tested.

## Language and version, with framework and package manager

| Component | Choice | Responsibility |
| --- | --- | --- |
| Host and LLM | Claude Code with the student's own Claude plan. We develop and test with Claude Sonnet 5.5. | Reads pages and résumés. Chooses tools and talks to the student. |
| Plugin format | Claude Code plugin with `.claude-plugin/plugin.json`, `skills/<name>/SKILL.md` and `.mcp.json` | Packages the skills and the MCP server |
| Browser | Claude in Chrome extension, started with `claude --chrome` | Reads the page and fills fields. Also uploads the résumé file |
| MCP server language | TypeScript 5.x with `@types/node`, on Node.js 22.13 or later, ES modules | Our own code |
| MCP framework | `@modelcontextprotocol/sdk`, stdio transport | Exposes typed tools to Claude |
| Validation | Zod 3 | Schemas for tool inputs and stored records. Rules too |
| Local database | Built-in `node:sqlite` module | Profile and answers. Personal rules and the run log too |
| Bundler | esbuild, one file at `server/dist/index.js`, committed | Teammates install the plugin without running `npm install` |
| Package manager | npm with a committed `package-lock.json`, installed with `npm ci` | Reproducible installs |

## Cloud provider and deployment model

The product runs locally. There is no cloud deployment and no server of our own. Each student installs the plugin in Claude Code, and Claude Code starts our MCP server on the laptop over stdio when a session begins. Developers load the working copy with `claude --plugin-dir ./plugins/job-application-assistant --chrome`.

Two hosted services are used. Claude itself runs on Anthropic's service under the student's plan. Approved shared rules are public JSON files in this repository under `rules/approved/`. The MCP server reads them over HTTPS from GitHub at startup and keeps the last good copy in its data folder, so it works offline with cached rules.

## Plugin layout

```text
plugins/job-application-assistant/
  .claude-plugin/plugin.json      name, version, description, author
  .mcp.json                       starts server/dist/index.js with node
  skills/
    set-up-profile/SKILL.md       résumé to confirmed records
    fill-application/SKILL.md     plan, batch fill, read back, review
    teach-a-fix/SKILL.md          demonstration to candidate rule
  server/
    src/  index.ts db.ts profile.ts answers.ts plan.ts rules.ts sanitize.ts sync.ts
    test/ *.test.ts
    dist/index.js                 bundled output, committed
rules/approved/index.json         list of approved rule files with versions
rules/approved/*.json             shared rules, public
.github/workflows/rules.yml       CI that tests every rule pull request
fixtures/forms/                   three static fixture forms
fixtures/applicants/              three synthetic applicants
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
        "JOBAPP_DB": "${CLAUDE_PLUGIN_DATA}/jobapp.db",
        "JOBAPP_RULES_URL": "https://raw.githubusercontent.com/Mikey-He/5940-Project/main/rules/approved/index.json"
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
| Approved shared rules | `rules/approved/` in this repository, cached locally | Hold steps and profile references only |
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
| `plan_fill` | Page URL and fingerprint. A list of fields, each with its label and type plus options and an element reference | Matches fields to profile facts and approved rules. Returns a plan plus the fields left for Claude |
| `record_fill_result` | Run ID and read-back values | Compares them with the plan and flags mismatches. Disables a failing rule locally |
| `find_answers` | Question text and word limit | Returns scoped saved answers that fit the limit |
| `save_answer` | Answer text and scope | Stores an approved answer |
| `propose_rule` | Steps from a demonstration | Validates the steps and replaces personal values with profile references. Saves the result as a personal rule |
| `export_rule_candidate` | Rule ID | Runs the personal-data scan, then writes a JSON file for a pull request after the student agrees |
| `sync_rules` | none | Downloads the approved rule index and verifies each rule against the schema. Caches the valid ones |

## Filling an application

1. The fill skill asks Claude to read the page structure once and list every field with its label and type, plus any options.
2. Claude calls `plan_fill`. Code fills what it can match exactly. It attaches an approved rule to any control the rule's fingerprint matches.
3. Claude decides only the fields code could not place, using the label and its section. For a file field, Claude calls `get_document` and uploads that file through Claude in Chrome.
4. Claude executes the plan in one batch with the browser's form tools and then reads every value back.
5. Claude calls `record_fill_result` and shows the review list. The student checks it and submits the application personally.

Claude in Chrome pauses at login pages and CAPTCHAs for the student. The skill never clicks a submit button.

## How a rule is learned and shared

A rule is a small JSON document with a target fingerprint and a list of steps. Values are profile references such as `{{profile.education[0].school}}`. A step must be one of these five types.

- `type`
- `select_option`
- `click`
- `wait_for`
- `read_back`

Validation has three layers.

1. **Schema.** Zod rejects unknown step types and literal values. The personal-data scan rejects anything that looks like personal data. That includes email addresses and phone numbers, plus any value found in the synthetic profiles.
2. **Replay.** A GitHub Actions workflow runs on every pull request that touches `rules/`. Playwright opens each fixture form, and a small test executor runs the rule for all three synthetic applicants. The pull request merges only if every value reads back correctly and a teammate approves.
3. **Runtime.** Each filled value is read back. A mismatch disables that rule on the laptop and flags the field. A fingerprint mismatch skips the rule.

### The shared rule folder

Shared rules live in this same GitHub repository, so we run no extra service. The repository is public, and the plugin reads it without signing in.

- `rules/approved/index.json` lists every approved rule file with its version. The plugin downloads this index at startup and then fetches only the files it has not cached.
- A student with a GitHub account opens a pull request with the exported file. A student without one sends the file to a teammate, who opens the pull request.
- To withdraw a bad rule, a teammate removes it from the index in a pull request. Every install drops it at the next start.

## Security basics and prohibited choices

- Page text and résumé text are data, not instructions, and so are job descriptions. Skills tell Claude to ignore instructions found in them. Text on a page can never add a tool call that the student did not ask for.
- Every MCP tool validates its input with Zod and refuses anything that fails.
- The plugin never submits an application or enters a password. It never creates an account or answers a sensitive question from a rule.
- No secrets exist in this product. No API keys are stored, because Claude Code handles the model connection under the student's login.
- Logs record tool names and outcomes with timings, never profile values.
- Only synthetic data is committed. `.gitignore` excludes `node_modules/` and `*.db`, and it also excludes `.env`.

| Prohibited | Reason | Use instead |
| --- | --- | --- |
| `better-sqlite3` or other native database modules | A native binary breaks the single bundled file on other operating systems | Built-in `node:sqlite` |
| Puppeteer or Selenium inside the plugin | Browser control must happen through Claude in Chrome, in the student's visible browser with its permission prompts | Claude in Chrome tools. Playwright is allowed only in tests |
| OpenAI or Anthropic SDKs inside the plugin | Claude Code already provides the model, and an SDK would need an API key | Skills and tool calls in the host Claude session |
| `axios` or `node-fetch` | Node 22 has `fetch` built in, so the extra dependency is unnecessary | Global `fetch` |
| Unofficial ATS or LinkedIn scraping libraries | They break site terms, and our MVP does not need them | Fixture forms and the student's open page |
| `eval` or `new Function` on rule content | A shared rule would become executable code | Interpret the fixed step types only |

## Test framework

| Layer | Tool | What it covers |
| --- | --- | --- |
| Unit | Vitest | Field matching and the planner. Rule schema and the personal-data scan. Answer scope |
| Rule replay | Playwright Test with the three fixture forms | Every shared rule fills every synthetic applicant correctly |
| Plugin checks | `claude plugin validate --strict` | Manifest and MCP entry, including paths |
| Scenario check | Manual run with Claude Code and Chrome, recorded | The five success criteria in `vision.md` |

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
      return { ref: field.ref, action: "ask_claude", reason: "searchable control without an approved rule" };
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

  it("hands a searchable control to Claude when no approved rule exists", () => {
    const [p] = planFill([school], profile, [], "fp-workday-1");
    expect(p.action).toBe("ask_claude");
  });

  it("uses an approved rule only when the page fingerprint matches", () => {
    const rule = { id: "r-school", fingerprint: "fp-workday-1", profileKey: "education.0.school", steps: [] };
    expect(planFill([school], profile, [rule], "fp-workday-1")[0].action).toBe("rule");
    expect(planFill([school], profile, [rule], "fp-workday-2")[0].action).toBe("ask_claude");
  });

  it("never guesses a field the profile does not cover", () => {
    const [p] = planFill([{ ref: "f9", label: "Favorite color", type: "text" }], profile, [], "fp");
    expect(p).toMatchObject({ action: "ask_claude" });
  });
});
```

## Reference documentation

- [Claude Code plugin manifest reference](https://code.claude.com/docs/en/plugins-reference)
- [Claude Code with Chrome](https://code.claude.com/docs/en/chrome)
- [Plugins in Cowork](https://support.claude.com/en/articles/13837440-use-plugins-in-cowork)
- [Model Context Protocol TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk)
- [AI-DLC v1.0.1](https://github.com/awslabs/aidlc-workflows/releases/tag/v1.0.1)
