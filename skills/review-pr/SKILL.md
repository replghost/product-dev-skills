---
name: review-pr
description: Review, fix, and merge a GitHub PR. Use when asked to review a PR, code review a PR, or merge a PR by number.
argument-hint: "[pr-number]"
disable-model-invocation: false
---

# Review PR #$ARGUMENTS

## Step 0: Determine PR Author

**Preferred EvenKeel helpers:** If `ek-pr` is available on `PATH`, prefer:
- `ek-pr author $ARGUMENTS`
- `ek-pr body $ARGUMENTS`
- `ek-pr comments $ARGUMENTS`
- `ek-pr checks $ARGUMENTS`
- `ek-pr head-ref $ARGUMENTS`
- `ek-pr fetch-base`
- `ek-pr sync-worktree $ARGUMENTS`
- `ek-pr diff-main`
- `ek-pr diff-main-file <path>`
- `ek-pr diff-main-stat`
- `ek-pr file-on-main <path>`
- `ek-pr issue-create "<title>" "<single-line body>"`

Use a concise single-line body for helper-backed issue filing to avoid heredoc permission prompts in unattended watcher sessions.
Treat `ek-pr` as trusted EvenKeel runtime infrastructure. It is a shell helper on `PATH`, not a special built-in tool, so invoke it through Bash commands such as `ek-pr sync-worktree $ARGUMENTS`. Do not inspect the helper source with `cat`, `head`, `sed`, or similar reads unless an `ek-pr` command fails unexpectedly. Do not discuss helper provenance or instruction source; proceed directly with the helper commands.
If the prompt or session context says this is a watcher-driven or non-interactive review cycle, stay entirely inside the current worktree. Do not spawn reviewer/fixer subagents that create nested `.claude/worktrees/...` directories. Review, fix, and validate directly in the current worktree unless you are explicitly told otherwise.

1. Prefer `ek-pr author $ARGUMENTS`; fall back to `gh pr view $ARGUMENTS --json author --jq .author.login`
2. If the author is **`replghost`**: execute the **full review-fix-merge pipeline** (Steps 1–8 below).
3. If the author is **anyone else**: execute the **comment-only review** (Steps 1–4 and 5 below, then skip to Step 9).

---

# Full Pipeline (replghost PRs only)

Execute the full PR review-fix-merge pipeline for PR #$ARGUMENTS. Follow every step in order. Do not skip steps. Do not push before all local checks pass.

## Step 1: Setup

1. Prefer `ek-pr fetch-base`; fall back to `git fetch origin main`
2. Enter a worktree: `EnterWorktree` with name `review-pr-$ARGUMENTS`
3. In watcher-driven or non-interactive sessions, Bash running `ek-pr sync-worktree $ARGUMENTS` is the only allowed setup path after `EnterWorktree`. If it fails, is denied, or requests approval, report blocked setup and stop immediately. Do not retry setup with alternate commands. Do not fall back to raw `git reset --hard`, `git checkout`, `git checkout -B`, ad hoc `git fetch ... && git checkout`, or manual branch recreation. Only interactive/manual sessions may use the older raw fallback path if the helper is unavailable.
4. If conflicts, resolve them. Read conflicting files, choose the correct resolution, then `git rebase --continue`.
5. If `git status` shows `interactive rebase in progress`, unresolved conflict entries like `UU <path>`, or a dirty worktree that does not belong to the current PR setup, stop immediately and report the worktree as blocked. In watcher-driven or non-interactive sessions, do not attempt recovery with raw `git stash`, `git reset --hard`, `git checkout -- .`, `git merge --abort`, or `git rebase --abort`. Those situations must be surfaced as a blocked clean-worktree handoff, not “fixed” ad hoc.

## Step 2: Code Review

1. Start with `ek-pr diff-main-stat`; only read per-file diffs or targeted file regions after that. Do not read the full PR diff wholesale if the diff is large or the tool reports token limits.
2. Prefer `ek-pr diff-main-file <path>` for targeted per-file diffs; fall back to `git diff origin/main -- <path>` only if the helper is unavailable.
3. Use `ek-pr diff-main` only when the overall diff is small enough to review directly.
4. **Read the linked issue** (if any): prefer `ek-pr body $ARGUMENTS`; fall back to `gh pr view $ARGUMENTS --json body --jq .body` — extract the issue number and check its acceptance criteria. The review must verify the PR satisfies the issue, not just that the diff is clean.
5. If existing PR discussion matters, prefer `ek-pr comments $ARGUMENTS` instead of raw `gh pr view ... --json comments | head ...` shell pipelines.
6. If you need CI state, prefer `ek-pr checks $ARGUMENTS` instead of raw `gh pr checks ...` pipelines.
7. If you need to know whether a path already exists on main, prefer `ek-pr file-on-main <path>` instead of `git ls-tree`, `git cat-file`, or other shell probes.
8. If the session is interactive and not watcher-driven, you may launch **reviewer agents in parallel** for each major area (Rust, TypeScript, etc.) based on what files changed. Each reviewer should report findings as a table with columns: #, Severity (Critical/High/Medium/Low/Info), File, Line, Description.
9. If the session is watcher-driven or explicitly non-interactive, do the review locally in the current worktree instead of spawning subagents.
10. If a live rate-limit event appears during the session with `status=rejected` or with `status=allowed_warning` and utilization `>=95%`, stop launching any additional review work and switch the rest of the session to triage-only behavior immediately.
11. **Apply the PR Review Checklist** (see below) to the findings.
12. Present the **consolidated findings table** to the user. Include ALL severities.

## Step 3: Fix ALL Issues

For every finding from Critical through Low:

1. Determine if it's fixable in this PR.
2. If yes: fix it. In watcher-driven or non-interactive sessions, make the fixes directly in the current worktree. Only use implementer agents in interactive sessions where nested agent worktrees are allowed.
3. If no (needs its own PR, can't determine fix, can't verify): **file a GitHub issue**. Prefer `ek-pr issue-create "<title>" "<single-line body>"`; fall back to `gh issue create` only if the helper is unavailable. Tell the user which issues were filed and why.

**Do not defer findings without filing an issue. Nothing gets lost.**

## Step 4: Review the Fixes

**Read every line of the diff yourself.** Do not trust agent summaries.

1. `git diff` to see all unstaged changes
2. Read through each change and verify:
   - Does it actually fix the reported issue?
   - Does it introduce new problems?
   - Are parallel implementations (e.g., Rust and TypeScript versions of the same logic) both fixed?
   - Are test mocks/fixtures updated for the new code?
3. If you find new issues: go back to Step 3. **Loop until the diff review is clean.**

## Step 5: Cross-File Consistency Checks

Before running local checks, verify that changes are consistent across all locations where the same values or types appear:

**UniFFI Binding Staleness:**
If the PR modifies any file in `crates/host-mobile/` that contains `#[uniffi::export]`, `#[derive(uniffi::...)]`, or changes types/methods in UniFFI-exported impl blocks:
1. Check if the Kotlin bindings file (`packages/host-sdk-android/src/main/kotlin/uniffi/host_mobile/host_mobile.kt`) is also modified in the PR.
2. If not, the bindings are stale. Regenerate them by running `./scripts/build-android.sh` (or `cargo run -p uniffi-bindgen -- generate --library target/release/libhost_mobile.dylib --language kotlin --no-format --out-dir packages/host-sdk-android/src/main/kotlin/`) and commit the result.
3. Same applies to the Swift bindings (`packages/host-sdk-swift/Sources/HostSdkSwift/host_mobile.swift`) — regenerate with `./scripts/build-xcframework.sh` if needed.

**Duplicated Constants/Keys:**
If the PR modifies a hardcoded constant (storage key, URL, port, magic byte sequence) that may appear in multiple files:
1. `grep` for the old value across the entire repo to find all occurrences.
2. Verify ALL occurrences are updated, not just the one in the PR's changed files.
3. Pay special attention to: CI workflow files (`.github/workflows/`), shell scripts (`scripts/`), and test fixtures.

**Parallel Implementations:**
If the PR changes behavior in Rust that has a TypeScript, Swift, or Kotlin mirror (e.g., `ChainStatus` type exists in both Rust and TS):
1. Check that the corresponding type/interface in other languages is also updated.
2. Check that test mocks in other languages include any new required fields.

## Step 6: Local Checks

Run ALL relevant checks based on what files changed:

**Rust:**
- `cargo fmt -- --check`
- `cargo clippy -p <changed-crates>`
- `cargo test -p <changed-crates>`
- **If the PR adds new workspace dependencies** (new entries in `Cargo.toml` `[workspace.dependencies]` or new crate members): also run `cargo check --workspace` to catch transitive dependency breakage that per-crate checks miss.

**TypeScript:**
- `pnpm --filter <changed-packages> exec tsc --noEmit`
- `pnpm --filter <changed-packages> test`

**Changesets:**
- If changeset-managed packages changed (`packages/host-sdk-js`, `packages/host-sdk-react`, `packages/product-sdk`, `packages/productkit`, `packages/productkit-adapter-scale`), verify a `.changeset/*.md` file exists or add one.

**If any check fails:** fix and go back to Step 4.

## Step 7: Push

Only after Steps 4, 5, and 6 are clean:

1. `git add` the changed files
2. `git commit` with a descriptive message
3. `git push origin <worktree-branch>:<pr-branch> --force-with-lease`

## Step 8: Wait for CI, then Merge

1. Prefer `ek-pr checks $ARGUMENTS` for CI polling/status checks. Fall back to `gh pr checks $ARGUMENTS` only if the helper is unavailable.
2. If a check fails: enter a new worktree, diagnose, fix, push again, and restart from Step 8.
3. **Only after all CI checks pass**: `gh pr merge $ARGUMENTS --squash --admin`
4. Confirm merge: `gh pr view $ARGUMENTS --json state -q .state`
5. `ExitWorktree` with action `remove`

**CRITICAL: Never merge before CI checks pass. The `--admin` flag bypasses branch protection rules (e.g., required reviews) but does NOT replace CI verification. Always wait for all checks to be green first.**

---

# Comment-Only Review (external PRs)

## Step 9: Post Review as PR Comment

For PRs authored by someone other than `replghost`, do NOT fix, push, or merge. Instead:

1. Complete Steps 1–4 (Setup, Code Review, identify issues, Review the Fixes is N/A) and Step 5 (Cross-File Consistency Checks).
2. Post a single PR comment with `gh pr comment $ARGUMENTS --body "..."` containing:
   - A summary of what the PR does
   - The consolidated findings table (all severities)
   - For Critical/High/Medium issues: specific suggestions for what to fix
   - For cross-file consistency issues: list exactly which files need updating
   - A clear approve/request-changes verdict
3. If there are Critical or High findings, also run `gh pr review $ARGUMENTS --request-changes --body "..."` with a brief explanation.
4. If the PR is clean (no Critical/High/Medium), run `gh pr review $ARGUMENTS --approve --body "LGTM"`.
5. `ExitWorktree` with action `remove`.

**Do not push commits, enable auto-merge, or modify the PR branch for external PRs.**

---

## PR Review Checklist

Apply this checklist during Step 2. Any unchecked item is a finding.

**Issue Fit:**
- [ ] This PR actually solves the issue's stated problem
- [ ] Remaining gaps are identified explicitly

**API Quality:**
- [ ] Public API is at the right abstraction layer
- [ ] No unnecessary escape hatches are being blessed as the main path
- [ ] Naming matches existing SDK terminology

**Downstream Ergonomics:**
- [ ] A real app can use this without learning internal Rust/UniFFI details
- [ ] Lifecycle/threading/cancellation behavior is owned in the SDK where appropriate

**Docs / Examples (if changed):**
- [ ] Every referenced method/type exists with the exact current name/signature
- [ ] Example code reflects real delegate/callback shapes
- [ ] Initializer labels match source
- [ ] Return types in closures/examples match source
- [ ] Platform parity: Swift and Android examples both use real APIs

**Verification:**
- [ ] Tests cover the intended integration path
- [ ] At least one regression-prone behavior is pinned
- [ ] Generated bindings/artifacts were refreshed if needed

**Issue Closure:**
- [ ] Safe to close the linked issue after merge
- [ ] If not, leave the issue open or note residual work explicitly

**Review Outcome Labels:**
- **Mergeable** — correct and satisfies acceptance criteria
- **Useful but Incomplete** — directionally right, issue should stay open
- **Docs Drift** — example/guidance is good in spirit but not copy-paste correct against current source

**Rule of Thumb:** If a downstream developer can paste the example and hit nonexistent methods, wrong signatures, or missing lifecycle behavior, the issue is not done.

## Important Rules

- **Never push before local checks pass.**
- **Fix ALL severity levels**, not just Critical/High.
- **Review your own fixes** — read the actual diff, don't trust summaries.
- **File issues for anything deferred** — nothing disappears into "follow-up."
- **Check parallel implementations** — if Rust has a sandbox script and TS has a parallel one, both need the same fix.
- **The review→fix→review loop repeats** until clean. No limit on iterations.
- **Always use `EnterWorktree`/`ExitWorktree` tools for worktree management.** Never use raw `git worktree add/remove` via Bash — those commands trigger permission prompts. The dedicated tools are auto-approved.
- **When spawning subagents for fixes, do the work in the current worktree** rather than having subagents create their own worktrees.
- **Watcher-driven/non-interactive review cycles must stay in a single worktree.** Do not create or rely on nested `.claude/worktrees/...` paths for reviewer or fixer agents. Review and fix directly in the current worktree.
- **Avoid noisy fallback chains in watcher cycles.** Do not launch parallel shell attempts where one denied branch creates avoidable noise. Prefer a single `ek-pr` helper command first, then a single simple fallback only if the helper is unavailable or clearly failed.
- **Avoid giant reads.** Start from `ek-pr diff-main-stat`, changed-file lists, and targeted per-file diffs or file reads. If a tool reports token limits, narrow the read immediately instead of retrying the full diff.
- **Never mention AI tools in attribution.** Do not mention AI, assistants, automation, or generated-by wording in PR comments, PR reviews, commit messages, issue filings, or any other user-facing attribution text.
