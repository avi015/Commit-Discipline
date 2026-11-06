---
name: commit-discipline
description: Enforces atomic, one-concern git commits with Conventional Commit messages, a why-focused body, and a two-gate human-approval checkpoint (plan, then commit) that applies even when the agent cannot reach the repository. Use when staging changes, composing a commit, or when the user asks to "commit", "write a commit message", "stage and commit", or "split these changes into commits".
tags: ["git", "commits", "conventional-commits", "atomic-commits", "version-control", "commit-message", "developer-workflow"]
license: MIT
---

# Commit Discipline

A small, opinionated skill for agents about to create a git commit. The goal is not to generate more code — it is to leave a history a human can read, review, bisect, and revert without guessing.

This skill governs **how** a commit is shaped. Whether committing is allowed at all is set by the user, not by this skill. If the user said "do not commit", hand the work back.

The gates, scope test, and secret procedure apply **whether or not the agent can run commands**. A no-repo session changes how work is delivered (runnable commands instead of executed ones), not whether the checkpoints are honored.

---

## The one-sentence test

Before staging anything, answer: **"What single change is this commit making?"** If the honest answer contains "and", it is not one commit.

| Answer | Verdict |
|---|---|
| "Adds rate limiting to the login endpoint." | One commit. |
| "Adds rate limiting and refactors auth and updates the README." | Three commits. |
| "Fixes the bug and adds a test." | One if the test proves the fix; two if unrelated. |
| "Various fixes." | Reject. Split by intent. |

**Every proposed commit carries its one-line summary**, shown at both gates. **A summary states the change, not the files.** "Updates the users API and its tests" restates the file list; "adds pagination to the users list endpoint" is a summary. If you cannot write the change without the diff, say so and ask for the diff.

### Scope test

If two parts of the change would need **two different scopes** in their subjects, they are **two commits** — not one `feat(users)` that quietly contains a UI change. If you are unsure whether a file is *required by* the change or merely *adjacent to* it, **ask**. An adjacent file does not ride along.

---

## Planning

### 1. Inventory

```bash
git status --short
git diff --stat
git diff --cached --stat
```

If commands cannot run, use the pasted `git status` and say so. Group by **intent**, not directory. Read the first column — it determines what remediation is possible:

| Status | Meaning | To stop it being committed |
|---|---|---|
| ` M` / ` D` | Unstaged | Just do not `git add` it. |
| `M ` / `A ` / `D ` | Staged | `git restore --staged <file>` |
| `M ` on a secret | In index, likely in history | `git restore --staged` + `git rm --cached` + `.gitignore` |
| `??` | Untracked | `.gitignore` only. |

### 2. Plan with summaries

```text
Proposed commits:
1. feat(auth): add rate limiting to login endpoint
   summary: adds per-IP rate limiting to the login route
   files: src/auth/rate-limit.ts, src/auth/login.ts
2. docs(auth): document rate-limit configuration
   summary: documents the new limit and window settings
   files: docs/auth.md
```

Apply the scope test while building the plan; split before presenting. Wait for plan acceptance before staging.

**When you cannot write a real summary yet** (no diff available, only pasted status):

- Use a **descriptive placeholder naming exactly what is missing**, per commit: `<which ESLint rule changed, and why>`.
- **Do not reuse a placeholder across commits.** Identical text means either the same concern or you have stopped thinking.
- **List every missing fact as a numbered question** so the user can answer in one reply.
- This is still Gate-1 approvable. A bare `<summary>` or a file-list restatement is a failure.

### 3. Stage per commit

Stage **named files only**. Never `git add .` or `git add -A`. If files were staged by an earlier bulk add, do not trust them — unstage strays by name and add the correct files by name.

**When handing back commands for more than one commit, mark each block:**

```bash
# ---- commit 1 goes here (after Gate 2) ----
git add src/api/users.ts src/api/users.test.ts
git status --short
```

Without the marker, a user running top to bottom stages every commit's files at once.

If a file looks like a secret, **stop and ask**. Do not stage it.

---

## Tracked-secret procedure

Unstaging does **not** remove a secret, and `.gitignore` does **not** untrack a file already in the index. For a file shown as `M ` or `A `, all six steps are required.

1. **Unstage**, leaving the working tree intact:

   ```bash
   git restore --staged <file>
   ```

2. **Check history:**

   ```bash
   git log --all --oneline -- <file>
   ```

3. **Report plainly:** no commits → staged but never committed, no rotation needed. Any commits → **the credential must be rotated**; anyone with a clone has it.

4. **Stop tracking it.** `.gitignore` is inert for a tracked file:

   ```bash
   git rm --cached <file>
   ```

   This must go in the **same commit** as the `.gitignore` change.

5. **Add to `.gitignore`** so a future `git add .` cannot re-add it.

6. **Offer history rewriting only if the user asks.** Purging is destructive and out of scope — hand it back rather than improvising `filter-repo` or `push --force`.

Never describe unstaging as "fixed," or a `.gitignore` entry as "stopping tracking." If commands cannot run, hand back the `git log --all` command and ask for the result before calling the file handled.

---

## Message format

```text
<type>(<scope>): <imperative summary>
```

`feat` new capability · `fix` bug fix · `refactor` neither · `docs` · `test` · `chore` build/deps/CI · `perf` · `style` formatting only.

Subject rules: imperative mood (`add`, not `added`) · lowercase after the colon · no trailing period · ≤72 characters total · specific and greppable.

**If you cannot tell whether a change is `feat` or `fix`, ask.** Guessing the type is the same sin as guessing the why.

The body answers **why**, not what: what the previous behavior was, what constraint forced this shape, what failure it prevents, what issue motivated it.

```text
feat(auth): add rate limiting to login endpoint

The login endpoint accepted unlimited attempts, allowing credential-
stuffing at ~2k requests/minute in staging. Limits each IP to 5 failed
attempts per 15 minutes; successful logins do not consume the budget.

Refs: SEC-214
```

Do not narrate the diff. If you do not know the why, do not invent it — use a **descriptive** placeholder and ask. Any placeholder means the **commit gate is closed**.

---

## The two gates

Two gates, two prompts. Do not use one for the other's job.

### Gate 1 — Plan approval

Use when any subject or body is still a placeholder, or the split is unconfirmed. Present the proposed commits with file lists, a per-commit summary (real or descriptive placeholder), and numbered questions for missing facts. End with:

> **Approve this plan? (yes / edit / split / cancel)**

A "yes" approves the **grouping only**. It authorizes no commit.

**Say this in your reply. Do not leave the user guessing what "yes" means.** Include a line such as: *"A 'yes' here approves the grouping only — it authorizes no commit; the commit prompt comes once each message is real text."*

### Gate 2 — Commit approval

Use only when the staged set has been shown, the full message is real text with no placeholders, and the summary is present. Present the staged file list, the full message, and the summary. End with:

> **Commit this? (yes / edit / split / cancel)**

Only this prompt authorizes a commit.

### Common rules

Explicit affirmative only — "looks fine" is not approval; ask again. Never commit silently, and never treat task completion as implicit permission. **edit** → revise and re-present. **split** → return to planning. **cancel** → leave the working tree as it was.

When commands cannot run, both gates still apply. Do not describe a handoff as complete while placeholders remain. Urgency does not waive either gate — the right answer to "I'm in a rush" is a **fast safe path** the user can approve in one reply, not a bulk commit.

---

## Execution and push

After Gate 2, with the staged set already shown:

```bash
git status --short            # confirm exactly the approved files are staged
git commit -m "<subject>" -m "<body>"
git log -1 --stat
```

If `git status --short` shows anything outside the approved list, **stop**: unstage the strays and re-present. Prefer the two-`-m` form over fragile shell quoting.

This skill does **not** push. A push is a separate action requiring separate confirmation — never chain `commit && push` unless asked for both. Before any push, run `git branch --show-current`; if it is `main` or `master`, offer a feature branch instead.

---

## Hard safety rules

| Never do this | Do this instead |
|---|---|
| `git commit --no-verify` | Fix the hook failure. Bypassing on a payments or auth change is the worst time to skip it. |
| `--amend` a pushed commit | Ask. Amend local, unpushed commits only, with approval. |
| `push --force` a shared branch | Never. Hand history changes back to the user. |
| Commit to `main` / `master` | Feature branch first, with approval. |
| `git add .` as reflex | Named files. |
| Commit with nothing staged | Stop. There is nothing to commit. |
| Bundle unrelated changes "for tidiness" | Split. Tidiness is not a reason to destroy reviewability. |
| Commit a staged set the user has not seen | `git status --short` first, every time. |
| Describe unstaging or a `.gitignore` entry as "handled" | Full tracked-secret procedure, including `git rm --cached`. |
| **Commit this?** while placeholders remain | **Approve this plan?** — you are at Gate 1. |
| Present a plan with no summary per commit | Include the one-sentence test result. |
| Write a summary that lists files | State the change, or ask for the diff. |
| Bare or reused `<summary>` | Per-commit descriptive placeholder + numbered questions. |
| Multi-commit commands with no markers | `# commit N goes here` between blocks. |
| Guess `feat` vs. `fix` | Ask. |
| Commit with merge conflict markers | Stop. Resolve conflicts first. |
| Commit from detached HEAD | Ask for a branch first. |
| End Gate 1 without saying "yes" approves grouping only | State it in the reply, not just in the skill. |

---

## When commands cannot run

If `git status` returns "not a git repository", or tool execution is unavailable: **do not pretend to have inspected the tree and do not claim a commit happened. Your first sentence must say plainly that the repository is not reachable from this session** — not "I haven't done anything yet".

Then:

1. Apply the one-sentence and scope tests to the pasted status.
2. Present the plan with a summary per commit (real or descriptive placeholder), note which files are already staged, and list missing facts as numbered questions.
3. Hand back named `git add` lines per commit, any `git restore --staged` / `git rm --cached` lines, **a `# commit N` marker between blocks**, and a `git status --short` step before each commit.
4. Close **Gate 1** with `Approve this plan? (yes / edit / split / cancel)` — not the commit prompt — and state that a "yes" approves the grouping only.
5. Once the user supplies the missing sentences, present each commit's staged list, full message, and summary, and close **Gate 2** with `Commit this? (yes / edit / split / cancel)`.
6. Tell the user to check `git branch --show-current` before pushing.

---

## Worked examples

### 1 — clean single commit (Gate 2)

```text
summary: adds per-IP rate limiting to the login route

feat(auth): add rate limiting to login endpoint

The login endpoint accepted unlimited attempts. Limit each IP to 5
failed attempts per 15 minutes; successful logins do not consume the
budget.

Refs: SEC-214
```

Then: **Commit this? (yes / edit / split / cancel)**

### 2 — tracked secret

`git status --short` shows `M  .env.local`.

1. `git restore --staged .env.local`
2. `git log --all --oneline -- .env.local` → if any commits, **rotate the credential**
3. `git rm --cached .env.local` — otherwise `.gitignore` will not untrack it
4. Add `.env.local` to `.gitignore`, in the **same commit**

```text
summary: stops tracking the local env file and ignores it going forward

chore: stop tracking .env.local

The local env file was tracked and is being removed from the index.
Existing history is unchanged; the credential must be rotated
separately if it was ever pushed.
```

Do not say "fixed" after unstaging, or commit it "just this once."

### 3 — no-repo session with a prior `git add .`

Status includes `M  .env.local`, `A  dist/bundle.js`, and legitimate edits.

1. **First sentence:** "I can't reach this repository from this session, so these commands have not been run."
2. Tracked-secret procedure on `.env.local` — hand back `git restore --staged`, `git log --all`, and `git rm --cached`; wait for the `git log` result.
3. `dist/bundle.js` is `A` (never committed) — unstaging is enough; add `dist/` to `.gitignore` in its own chore commit.
4. Plan with summaries (or descriptive placeholders) and numbered questions.
5. Commands with `# commit N` markers and `git status --short` per commit.
6. **Gate 1:** `Approve this plan? (yes / edit / split / cancel)` — plus the explicit line that a "yes" approves grouping only.
7. **Gate 2** once messages are real: staged list + message + summary → `Commit this? (yes / edit / split / cancel)`

---

## Limitations

- Team conventions (`CONTRIBUTING.md`, `commitlint`) take precedence; read them first.
- This skill does not push, rebase, rewrite history, or purge secrets. Those are separate operations with separate risks — hand them back.

---

## Stop rule

If the message would need "and" to be honest, if the staged set spans more than one intent, if the plan needs two scopes in one commit, if a commit has no summary or a bare/reused placeholder, if the staged set has not been shown, if a tracked secret was "handled" without `git rm --cached`, if multi-commit blocks have no markers, or if placeholders remain — **stop and replan**.

A missed commit is cheap. A tangled commit is expensive. A leaked secret is worse than both. Approving a plan is not approving a commit, and `.gitignore` is not untracking.