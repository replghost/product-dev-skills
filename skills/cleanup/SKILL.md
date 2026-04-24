---
name: cleanup
description: Safely free disk space by removing stale worktrees, build caches, and temp files. Checks for uncommitted work, unpushed branches, stashes, and open PRs before removing anything.
argument-hint: "[--dry-run] [--force]"
---

# Disk Space Cleanup

Free disk space safely. **Never delete anything the user might still be working on.**

## Step 1: Survey Disk Usage

1. `df -h /` to show current free space.
2. Measure each cleanup-candidate area:
   ```
   du -sh ~/dev/github/paritytech/host-sdk/target 2>/dev/null
   du -sh ~/dev/github/paritytech/host-sdk/.claude/worktrees 2>/dev/null
   du -sh ~/.codex/worktrees 2>/dev/null
   du -sh ~/dev/github/paritytech/host-sdk-worktrees 2>/dev/null
   du -sh ~/Library/Developer/Xcode/DerivedData 2>/dev/null
   du -sh ~/Library/Developer/CoreSimulator 2>/dev/null
   du -sh ~/.cargo/registry 2>/dev/null
   du -sh ~/.gradle/caches 2>/dev/null
   ```
3. Also scan the user's home directory for any other large directories:
   ```
   du -sh ~/dev/github/paritytech/host-sdk-* 2>/dev/null
   du -sh ~/.claude/worktrees 2>/dev/null
   ```
4. List all registered worktrees: `git worktree list`
5. Scan for unregistered worktree directories in `/private/tmp/host-sdk-*`

Present a summary table of areas and sizes.

## Step 2: Classify Worktrees

For **every** worktree directory (registered or unregistered), determine its status.

### Safety-First Principle

A worktree is **safe to remove ONLY if ALL of the following are true**:
- The branch exists on the remote (`git ls-remote --heads origin <branch>` returns output)
- No uncommitted changes
- No stash entries reference work from this worktree
- No open PR on this branch
- No recent activity (> 7 days)

If the branch is LOCAL-ONLY (no remote counterpart), the worktree is **NEVER safe to remove** because deleting it loses the branch and all commits permanently.

### In-Use Checks (ALL must pass for STALE classification)

1. **Branch exists on remote** — `git ls-remote --heads origin <branch>` returns output. If the branch does NOT exist on the remote, classify as **IN-USE (local-only branch)** immediately — do not proceed with other checks. Deleting this worktree would permanently lose commits.

2. **Uncommitted changes** — `git -C <path> status --porcelain` returns no output

3. **No stash entries** — Check the **repo-wide** stash (`git stash list`), not just per-worktree. Git stashes are shared across all worktrees of the same repo. If any stash entry references this worktree's branch, mark as IN-USE.

4. **No unpushed commits** — The local branch tip matches the remote branch tip:
   ```
   local=$(git -C <path> rev-parse HEAD)
   remote=$(git ls-remote --heads origin <branch> | cut -f1)
   ```
   If `$local != $remote`, mark as IN-USE (has unpushed local commits).

5. **No open PR** — `gh pr list --head <branch> --state open --json number -q '.[].number'` returns empty

6. **No active process** — `pgrep -f <worktree-path>` returns nothing; no tmux sessions reference the path

7. **No recent activity** — `git -C <path> log -1 --format=%ct` is older than 3 days

### Classification

- **STALE**: ALL safety checks pass. The branch is pushed to remote, tips match, no dirty state, no stashes, no open PR, no recent activity. Safe to remove — the work is fully preserved on the remote.
- **PRUNABLE**: Directory is already gone but git still tracks it (`prunable` in `git worktree list`). Safe to prune.
- **IN-USE**: One or more checks failed. **Do NOT remove.** State the reason.

Present a table with columns: Path, Branch, Size, Status (STALE/PRUNABLE/IN-USE), Reason.

## Step 3: Present Cleanup Plan

Show the user what will be cleaned and what will be kept:

```
WILL REMOVE (stale — branch exists on remote, tips match, clean state):
  <path> — <branch> — <size> — last activity <date>
  ...

WILL KEEP (in-use):
  <path> — <branch> — <size> — <reason>
  ...

BUILD CACHES (always safe to clear):
  cargo target/ — <size>
  Xcode DerivedData — <size>
  gradle caches — <size>
  ...

Estimated space freed: <total>
```

If `$ARGUMENTS` contains `--dry-run`: stop here, do not proceed to Step 4.

## Step 4: Confirm and Execute

**Wait for user confirmation before proceeding.** Unless `$ARGUMENTS` contains `--force`, ask: "Proceed with cleanup? (removes stale worktrees + build caches)"

Once confirmed:

1. **Remove stale worktrees** — `rm -rf <path>` for each STALE worktree only
2. **Prune git references** — `git worktree prune -v`
3. **Clean build caches** (only if they exist and are > 500MB):
   - `cargo clean` (in the main repo)
   - `rm -rf ~/Library/Developer/Xcode/DerivedData/*`
   - `rm -rf ~/.gradle/caches/*`
4. **Do NOT touch**:
   - `~/.cargo/registry` or `~/.cargo/git` (these cache downloads, rebuilding is expensive)
   - `~/Library/Caches` (system-managed)
   - `~/Library/Developer/CoreSimulator` (may contain user data; suggest `xcrun simctl delete unavailable` instead)
   - Any IN-USE worktree
5. Show `df -h /` after cleanup to confirm space freed.

## Important Rules

- **A worktree with a local-only branch is NEVER safe to remove.** This is the most important rule. If `git ls-remote --heads origin <branch>` returns nothing, the worktree must be kept regardless of all other signals.
- **Even if a branch exists on remote, verify tips match.** A pushed branch with additional local commits still has unpushed work.
- **Git stashes are repo-wide, not per-worktree.** Check `git stash list` from the main repo and flag any worktree whose branch appears in a stash entry.
- **NEVER remove worktrees in bulk** — evaluate each one individually.
- **When in doubt, keep it.** A false negative (keeping something stale) wastes disk space. A false positive (deleting something in-use) loses work.
- **Log everything you remove** so the user can see exactly what was cleaned.
- **For local-only worktrees that are actually stale**, suggest pushing the branch to remote first (`git push origin <branch>`) so the worktree can be safely removed on the next cleanup run. Present these as a separate "PUSH FIRST, THEN REMOVE" list.
