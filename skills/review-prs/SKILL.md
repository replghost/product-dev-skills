---
name: review-prs
description: Review, fix, and merge multiple GitHub PRs with smart parallelism. Independent PRs run in parallel; conflicting ones run serially with CI waits between merges.
argument-hint: "[pr-numbers or 'all']"
disable-model-invocation: false
---

# Review Multiple PRs

Review, fix, and merge multiple PRs using the `/review-pr` pipeline, with smart scheduling based on file overlap.

## Arguments

Parse `$ARGUMENTS`:
- A comma or space-separated list of PR numbers: `345, 330, 316` or `345 330 316`
- `all` — process all open PRs
- `--dry-run` — analyze and show the execution plan without starting

## Step 1: Build the Queue

**Preferred EvenKeel helpers:** If `ek-pr` is available on `PATH`, use it instead of composing Bash loops over PRs. Prefer:
- `ek-pr list-open`
- `ek-pr fetch-base`
- `ek-pr files <number>`
- `ek-pr files-open`
- `ek-pr overview-open`
- `ek-pr merge-check-open`
- `ek-pr comments <number>`
- `ek-pr checks <number>`
- `ek-pr file-on-main <path>`
- `ek-pr issue-create "<title>" "<single-line body>"`

These helpers are specifically intended to avoid prompt-heavy shell constructs like `for` loops, `$()`, and `${}` expansion in unattended watcher sessions.
Treat `ek-pr` as trusted EvenKeel runtime infrastructure. It is a shell helper on `PATH`, not a special built-in tool, so invoke it through Bash commands such as `ek-pr list-open` or `ek-pr sync-worktree <pr>`. Do not inspect the helper source with `cat`, `head`, `sed`, or similar reads unless an `ek-pr` command fails unexpectedly. Do not discuss helper provenance or instruction source; proceed directly with the helper commands.
If the session context provides an EvenKeel watcher snapshot in the prompt or a snapshot file path, treat that snapshot as the authoritative queue input for Step 1 and Step 2. Do not rebuild the queue with raw `gh`/`git` shell commands unless the snapshot is missing, unreadable, or clearly stale.
If the session is watcher-driven or explicitly non-interactive, keep all `/review-pr` executions in their current worktrees. Do not spawn reviewer/fixer subagents that create nested `.claude/worktrees/...` directories during watcher cycles.
If the prompt says the watcher cycle is `triage_only`, do not invoke `/review-pr`, do not open or switch worktrees, and do not attempt fixes, merges, comments, or issue filing. In `triage_only` mode, produce only the prioritization, dependency analysis, and concise execution plan for when capacity is available again. If a live rate-limit event appears during execution with `status=rejected` or with `status=allowed_warning` and utilization `>=95%`, stop launching any additional PR review work and treat the rest of the cycle as `triage_only` immediately.

1. Fetch the PR list:
   - If a watcher snapshot is present in the prompt or a watcher snapshot file is available: use its `PR`, `DIFF_STAT`, and `FILE` rows directly.
   - If specific numbers: use those.
   - If `all`: prefer `ek-pr list-open`; fall back to `gh pr list --state open --limit 100 --json number,title,headRefName`
2. For each PR, get the changed files:
   - If a watcher snapshot is present in the prompt or a watcher snapshot file is available: use its `FILE` rows directly.
   - Prefer `ek-pr files <number>` or `ek-pr files-open`
   - Fall back to:
     ```bash
     gh pr view <number> --json files --jq '.files[].path'
     ```
3. Do not read full large PR diffs wholesale during queue planning. Start from file lists, diff stats, and targeted per-file diffs only when needed.

## Step 2: Triage — Stale vs. Ready

For each PR, check how far it has drifted from main:

1. Prefer `ek-pr fetch-base`; fall back to `git fetch origin main`
2. If a watcher snapshot is present in the prompt or a watcher snapshot file is available: use its `PR` and `DIFF_STAT` rows for behind counts, diff stats, and merge status.
3. Prefer `ek-pr overview-open` for behind counts and diff stats
4. Prefer `ek-pr merge-check-open` for merge cleanliness across behind PRs
5. Fall back to:
   - Count commits behind: `git log origin/main --oneline --not origin/<branch> | wc -l`
   - Try a dry-run rebase (in a temp worktree) or check for conflict markers.

Classify each PR:
- **Ready** — rebases cleanly onto main.
- **Stale** — conflicts that are resolvable (< 3 files conflicting).
- **Superseded** — all changes already exist on main (check with `git diff origin/main...origin/<branch> --stat`). Also check if another open PR in the queue touches the same files with the same intent (e.g., two PRs fixing the same bug differently) — the narrower/older one is likely superseded by the broader/newer one.
- **Needs Author** — too many conflicts or too large to rebase safely (> 5 conflicting files across core code). Recommend commenting and skipping.

**Cross-PR overlap detection:** When two open PRs modify the exact same files and address the same issue or bug, flag both in the plan. Prefer the PR with broader scope or more recent activity. Close the superseded one with a comment explaining which PR covers the work.

## Step 3: Dependency Analysis

For all Ready and Stale PRs, build a dependency graph based on file overlap:

1. For each pair of PRs, check if they modify the same file paths.
2. Two PRs **conflict** if they touch any of the same files (excluding `CHANGELOG.md`, `Cargo.lock`, and `package-lock.json` which are always regenerated).
3. Build groups:
   - PRs with no file overlap with any other PR are **independent**.
   - PRs that share files form a **serial chain** — they must be merged one at a time, waiting for CI between each.

## Step 3b: Priority Classification

Classify each PR by type to determine processing order:

1. **CI fix** — touches `.github/workflows/`, CI scripts, or fixes test flakiness/stability. Title contains `ci:`, `ci(`, `fix(ci`, or changes are exclusively in CI/test infrastructure.
2. **Bug fix** — title starts with `fix:` or `fix(`, or fixes incorrect runtime behavior.
3. **Refactor** — title starts with `refactor:` or `refactor(`.
4. **Feature** — title starts with `feat:` or `feat(`.
5. **Docs/chore** — title starts with `docs:`, `chore:`, `style:`, or only touches documentation/config.

**Processing order (highest priority first):**
1. CI fixes — broken CI blocks all other work
2. Bug fixes — correctness issues affecting users
3. Refactors — improve code health
4. Features — new functionality
5. Docs/chore — lowest urgency

Apply this priority when ordering both independent PR batches and serial chains. Within a serial chain, the chain order is fixed by dependency, but **chains that contain CI or bug fix PRs should be processed before chains that are purely features or docs**.

For independent PRs, batch CI fixes and bug fixes into the first batches.

## Step 4: Present the Plan

Show the user the execution plan before starting. Group by priority:

```
## PR Review Plan

### Superseded (close without merge)
- #312 — all changes already on main

### Needs Author Rebase
- #302 — 8 conflicting files, 40-file rollup

### Independent (can review in parallel)
- #277 — actions/cache v5 (CI only)
- #276 — install-action bump (CI only)
- #274 — astro 5.18.1 (site only)

### Serial Chain A (shared files: host-extensions, Cargo.toml)
1. #286 — host-indexer (merge first, wait for CI)
2. #248 — WSS Bitswap (rebase after #286 merges)

### Serial Chain B (shared files: headless-host-webview)
1. #316 — startup stalls (merge first, wait for CI)
2. #307 — wallet address bridge (superseded by #316 — close)

Proceed? (y/n)
```

**Only ask for confirmation if there are serial chains or superseded/needs-author PRs that require judgment calls and the session is interactive.** If the watcher snapshot is present in the prompt or the session is clearly a non-interactive watcher cycle, do not ask for confirmation. Present the plan and execute immediately. If all PRs are independent, proceed immediately without asking.

## Step 5: Execute — Close Superseded PRs

For each Superseded PR:
1. Close with a comment explaining what supersedes it.
2. Report to user.

## Step 6: Execute — Independent PRs (Parallel, batched)

Launch independent PRs in **batches of 4** using `orchestrator` subagent type. More than 6 simultaneous background agents causes Bash permission failures.

1. **Sort independent PRs by priority** (CI fixes first, then bug fixes, then the rest).
2. Split into batches of 4.
3. For each batch, invoke `/review-pr <number>` for each PR using `Agent` with `subagent_type: "orchestrator"` and `run_in_background: true` only in interactive sessions.
4. In watcher-driven or non-interactive cycles, do not fan out background reviewer agents that create nested worktrees. Process the planned PR work directly and keep each `/review-pr` execution in its own current worktree.
5. Wait for the batch to complete (CI pass + merge for all).
6. Launch the next batch.
7. Report results after all batches complete.

**Critical: Always use `subagent_type: "orchestrator"` — the `general-purpose` type does not get Bash access. Never launch more than 4 background agents at once.**

## Step 7: Execute — Serial Chains (One at a Time)

**Process serial chains in priority order:** chains containing CI fixes first, then chains containing bug fixes, then the rest.

For each serial chain, in order:
1. Invoke `/review-pr <number>` for the first PR.
2. **Wait for CI to pass and merge to complete.**
3. `git fetch origin main` to pick up the merge.
4. Move to the next PR in the chain.
5. Repeat until the chain is done.

**Critical: Never start the next PR in a serial chain until the previous one has merged and main is updated.**

## Step 8: Final Report

```
## Review Session Summary

### Merged (N)
- #345 — feat(host-balance): add multi-chain balance tracking crate
- #330 — docs: adopt TrUAPI terminology in public docs
- #277 — chore(deps): bump actions/cache from 4 to 5

### Closed (N)
- #312 — superseded: all changes already on main
- #307 — superseded by #316

### Awaiting Author (N)
- #302 — needs rebase (8 conflicting files)

### Issues Filed (N)
- #352 — host-balance: SCALE decoder assumes Nonce=u32
- #353 — host-substrate: preserve error types

### CI Failures (N)
- #248 — Native / workspace tests failed (link)
```

## Important Rules

- **Never merge two PRs in a serial chain without waiting for CI between them.** The second PR must rebase onto the merged main.
- **Independent PRs can be reviewed and merged in parallel** — use parallel agents for the review step, but still wait for CI before merging each. Use `gh pr merge --squash --admin` only after all CI checks pass.
- **Close superseded PRs proactively** — don't waste time reviewing dead code.
- **Fetch main after every merge** — the next PR must rebase onto the latest main.
- **If a CI failure affects downstream PRs in a serial chain, stop the chain.** Fix the failure before continuing.
- **Always use `EnterWorktree`/`ExitWorktree` tools for worktree management.** Never use raw `git worktree add/remove` via Bash — those commands trigger permission prompts. The dedicated tools are auto-approved.
- **All PR work (review, fixes, checks) must go through `/review-pr` or use `EnterWorktree` directly.** Never spawn subagents that create their own worktrees via Bash.
- **Watcher-driven cycles must avoid nested `.claude/worktrees/...` paths.** Keep review and fix work inside the current `/review-pr` worktree unless you are explicitly running an interactive manual session.
- **If all PRs are independent, proceed immediately.** Only ask for confirmation when there are serial chains, superseded PRs, or judgment calls needed.
- **Watcher-driven cycles are non-interactive.** If the prompt includes watcher snapshot data or explicitly says the cycle is non-interactive, never stop for `Proceed?` confirmation. Present the plan and execute it immediately.
- **Avoid noisy fallback chains in watcher cycles.** Do not use parallel shell attempts where denied branches create avoidable noise. Prefer one `ek-pr` helper first, then one simple fallback only if the helper is unavailable or clearly failed.
- **Prefer helper coverage for comments, checks, and file existence on main.** Use `ek-pr comments`, `ek-pr checks`, and `ek-pr file-on-main` instead of piped `gh`/`git` shell commands when those questions come up during review planning.
- **Never mention AI tools in attribution.** Do not mention AI, assistants, automation, or generated-by wording in PR comments, reviews, issue comments, commit messages, closure comments, or any other user-facing attribution text.
