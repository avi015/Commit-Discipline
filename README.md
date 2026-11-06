<img src="Commit%20Discipline%20Tech%20Badge.png" width="30%">
# 🛡️ Commit Discipline

> Enforces atomic, one-concern git commits with Conventional Commit messages, why-focused bodies, and a strict two-gate human-approval workflow.

![Type](https://img.shields.io/badge/type-agent--skill-blue)
![Format](https://img.shields.io/badge/format-SKILL.md-orange)
![Standard](https://img.shields.io/badge/standard-Conventional%20Commits-green)
![License](https://img.shields.io/badge/license-MIT-blue)

**Tags:** `git` `commits` `conventional-commits` `atomic-commits` `version-control` `commit-message` `developer-workflow` `agent-skill` `skill-md` `claude-code` `cursor` `aider` `codex` `gemini-gems` `claude-projects` `prompt-engineering`

---

## Overview

AI coding agents often bundle unrelated changes together, generate generic or file-restating commit messages, and bypass careful verification before writing to version control. The **Commit Discipline** skill provides a rigorous protocol governing how git commits are staged, structured, and approved. By enforcing the "one-sentence test" and the scope test, it splits multi-concern diffs into atomic commits, formats messages to document *why* a change occurred rather than *what* changed, and demands explicit human confirmation through two separate approval gates.

| Output Component | Rules & Enforced Standards |
|---|---|
| **Inventory & Commit Plan** | Evaluates files by intent rather than directory; splits changes requiring different scopes; includes an imperative one-line summary per proposed commit. |
| **Staging Commands** | Requires explicit file paths (`git add <file>`); strictly bans bulk staging (`git add .` or `-A`); marks sequential commit blocks clearly. |
| **Commit Message** | Follows Conventional Commits (`type(scope): imperative summary`); subject $\le 72$ chars, lowercase post-colon, no period; body explains historical context, constraints, and motivation. |
| **Approval Checkpoints** | Two distinct checkpoints: Gate 1 approves grouping only; Gate 2 approves actual commit execution once no placeholders remain. |
| **Secret Remediation** | Mandatory 6-step procedure (`git restore --staged`, `git log --all`, `git rm --cached`, and `.gitignore`) whenever credentials or `.env` files are tracked. |

Built-in guardrails prevent committing directly to `main` or `master`, amending pushed commits, skipping git hooks with `--no-verify`, silent committing, and bundling unrelated edits for superficial tidiness.

---

## Features

- ✅ **Strict Atomic Commit Boundaries:** Applies the one-sentence test to reject bundled changes that require an "and" to summarize.
- ✅ **Two-Gate Human Verification:** Isolates plan confirmation (Gate 1) from commit authorization (Gate 2) to eliminate accidental execution.
- ✅ **Why-Focused Commit Bodies:** Enforces commit descriptions that record prior behavior, technical constraints, and root causes instead of diff narration.
- ✅ **Tracked-Secret Protocol:** Detects staged or tracked credentials and mandates cache removal, history checks, and ignore rules.
- ✅ **No-Repo Graceful Degradation:** Operates transparently when repository command execution is disabled by generating labeled, copy-pasteable shell blocks.
- ✅ **Branch and Safety Defenses:** Refuses to commit to detached HEADs or protected branches without explicit direction to branch off first.

---

## Installation

### Claude Code

Install the skill for an individual repository or globally across your environment:

```bash
# Project-level install
mkdir -p .claude/skills/commit-discipline
cp SKILL.md .claude/skills/commit-discipline/

# User-level install (available across all projects)
mkdir -p ~/.claude/skills/commit-discipline
cp SKILL.md ~/.claude/skills/commit-discipline/
```

Claude Code discovers skills automatically from the front matter description and activates `commit-discipline` when you ask to stage, split, or craft commit messages.

### Other Agentic Tools

| Tool | Suggested setup |
|---|---|
| **Cursor** | Copy the contents into `.cursor/rules/commit-discipline.mdc`, or reference it directly in chat using `@SKILL.md`. |
| **Aider** | Add the file to read-only memory during startup: `aider --read SKILL.md`. |
| **Codex & `AGENTS.md`-aware agents** | Paste the skill contents directly into `AGENTS.md` or instruct the agent to follow `SKILL.md`. |
| **Any other assistant** | Paste the contents of `SKILL.md` at the beginning of the chat conversation before initiating work. |

### Chat-Based Setups (No File System)

| Platform | Setup Instructions |
|---|---|
| **Claude Projects** | Create a project, upload `SKILL.md` under Project Knowledge, or paste into Project Instructions. |
| **Gemini Gems** | Create a Gem and paste the full skill text into the Instructions box or attach as a reference document. |
| **Custom GPTs** | Paste the contents of `SKILL.md` directly into the Instructions configuration panel. |

*Exact file locations and menu labels vary by tool and version, so check each tool's documentation if something doesn't load.*

---

## Usage

| Input Parameter | Description |
|---|---|
| `git status` / `git diff` output | The short status (`git status --short`) or unified diff of your current working directory. |
| Task intent / user prompt | High-level goal (e.g., "commit my recent payment bug fix and doc updates"). |
| Repository accessibility context | Indication of whether the agent has shell execution access or must emit manual bash snippets. |

### Example Prompt

```text
Use the commit-discipline skill.

Here is my current status:
 M src/billing/webhook.ts
 M src/billing/webhook.test.ts
 M docs/payments.md
?? .env.local

Help me stage and commit these changes.
```

In tools without native skill support, replace the first line with: "Follow the instructions in SKILL.md."

---

## Sample Input

```text
Use the commit-discipline skill to review and commit my changes. You cannot execute terminal commands directly in this session.

Here is my git status --short:
 M src/auth/session.ts
 M src/auth/session.test.ts
 M config/session.json
 M README.md
```

---

## Sample Output

I cannot reach this repository from this session, so these commands have not been run.

Applying the one-sentence test and scope tests to your status reveals two distinct intents across different scopes (`auth` and `docs`). Here is the proposed plan:

**Proposed commits:**
1. `feat(auth)`: add absolute session expiration timeout
   summary: enforces absolute session expiration after 8 hours
   files: `src/auth/session.ts`, `src/auth/session.test.ts`, `config/session.json`
2. `docs(readme)`: document session duration environment variables
   summary: updates configuration instructions for session TTL settings
   files: `README.md`

### Step 1: Stage and inspect the first commit

```bash
# ---- commit 1 goes here (after Gate 2) ----
git add src/auth/session.ts src/auth/session.test.ts config/session.json
git status --short
```

### Gate 1 Verification

> **Approve this plan? (yes / edit / split / cancel)**

*A "yes" here approves the grouping only — it authorizes no commit; the commit prompt comes once each message is real text.*

> ℹ️ The sample above is illustrative. Always review generated content for accuracy and compliance before using it.

---

## Customization

- **Conventional Commit Types:** Edit the *Message format* list to allow custom organization types (e.g., `build`, `infra`, `ui`) or enforce strict team schemas.
- **Subject Length Limit:** Adjust the 72-character upper limit in the subject format rules to match your team's `commitlint` rules.
- **Gate Prompts:** Customize the exact wording of the Gate 1 and Gate 2 challenge prompts to integrate with custom internal CLI wrappers.
- **Branch Protection Policy:** Modify the hard safety rule governing `main` / `master` to include branches like `develop`, `release/*`, or `staging`.

---

## Tips for Best Results

1. **Paste Raw `git status --short`:** Providing the two-column status output allows the agent to immediately spot untracked artifacts, unstaged changes, and staged index files.
2. **Provide Diff Context for Bodies:** If you want deep, why-centric commit bodies immediately, provide `git diff` excerpts alongside your status so the agent does not need to pause with descriptive placeholders.
3. **Respect Gate 1 Boundaries:** Do not treat approving a commit grouping as permission to run execution scripts; ensure each subject and body is free of placeholders first.
4. **Handle Secrets Instantly:** If an environment file appears as `M ` or `A `, follow the six-step sequence immediately to scrub it from git's index prior to staging application code.

---

## Disclaimer

This skill generates commit groupings and message drafts based on user-provided diffs and working tree statuses; project policies, CI linting rules, and branch conventions may vary. This skill is not affiliated with, sponsored by, or endorsed by GitHub, Conventional Commits, or any referenced agentic tool. You are solely responsible for inspecting staged sets and verifying commit messages before applying or pushing changes to remote repositories.

---

## Contributing and License

Contributions to improve commit safety, parsing logic, and agent integrations are welcome:
- Add detection heuristics for common API key and secret formats.
- Provide hook scripts to automate Gate 1 and Gate 2 verification in CI environments.
- Extend Conventional Commit parsing to support JIRA/Linear ticket prefix automation.

Distributed under the **MIT** License.
