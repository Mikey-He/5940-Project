# 5940-Project

## Team Members 
Liaba Gondal - lg695
Tifany Martinez - Tm783
Mikey He - ch2373
Chen Chen - cc3254
Eesha Kamran - ek843

## AI-DLC setup

This repository includes **AI-DLC v1.0.1** rules for **OpenAI Codex**, installed using the official [v1.0.1 Codex setup instructions](https://github.com/awslabs/aidlc-workflows/blob/v1.0.1/README.md#openai-codex).

- `AGENTS.md`: unmodified copy of the release's `core-workflow.md`.
- `.aidlc-rule-details/`: complete, unmodified detailed rules from the same release, committed to Git.
- Source: [ai-dlc-rules-v1.0.1.zip](https://github.com/awslabs/aidlc-workflows/releases/download/v1.0.1/ai-dlc-rules-v1.0.1.zip).
- Release archive SHA-256: `ac0601544b6c7ba41b541a7a96259d6ea3995ea0b94b2a58a333ae7898e22b39`.

Use v1.0.1 for this course; do not replace these rules with the upstream main branch or the latest release.

## Starting the team workflow

Clone or pull this repository and open a new Codex session with this repository as the project root so it loads `AGENTS.md`. To check that the rules are available, ask: "Using AI-DLC, describe the workflow available in this repository without starting it or changing any files."

Before starting Inception, the team must agree on the product and review the README, vision document, and technical environment document. These project-specific inputs still need to be prepared; installing the rules does not complete Inception.

Once the input documents are ready, start one shared team workflow with a prompt such as:

> Using AI-DLC, I want to start a new project. Read our vision document and technical environment document, then begin the Inception workflow. For this milestone, complete the required Inception stages through Units Generation and stop before Construction.

Identify the actual input document paths in that prompt. One teammate operates the agent while the team discusses answers, reviews artifacts, and approves each stage. Do not run separate Inception workflows for individual teammates.

The workflow creates `aidlc-docs/`, including `audit.md` and `aidlc-state.md`. Commit these records and the generated artifacts as the workflow progresses. Application code belongs outside `aidlc-docs/`. Never commit API keys or other secrets.
