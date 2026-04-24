---
name: work-issues
description: Loop through open GitHub issues and work them sequentially using the full development pipeline. Optionally filter by label or limit count.
argument-hint: "[optional: --label bug --limit 5]"
disable-model-invocation: false
---

# Work Through Open Issues

Process open GitHub issues sequentially using the `/work-issue` pipeline for each.

## Arguments

Parse `$ARGUMENTS` for optional flags:
- `--label <label>` — only process issues with this label (can be repeated)
- `--limit <n>` — max number of issues to process (default: no limit)
- `--skip <n1,n2,...>` — comma-separated issue numbers to skip
- `--dry-run` — list which issues would be worked on, but don't start

If no arguments provided, fetch all open issues.

## Step 1: Build the Queue

1. Fetch open issues:
   ```bash
   gh issue list --state open --limit 100 --json number,title,labels,assignees,author --author replghost
   ```
2. **Only include issues authored by `replghost`.** Skip issues filed by anyone else — those need manual review first.
3. Apply filters (label, skip list).
4. Exclude issues labeled `wontfix`.
5. Exclude issues that are already assigned to someone else.
6. **Priority sort.** Unless `--label` is provided (which implies the user already chose what to work on), sort issues by label-based priority tier, then by issue number ascending within each tier:
   - **Tier 1 (critical):** `security`
   - **Tier 2 (broken):** `bug`, `ci`
   - **Tier 3 (improvements):** `enhancement`, `testing`
   - **Tier 4 (everything else):** issues with no recognized priority label

   An issue's tier is determined by its highest-priority label. If `--label` is provided, skip this sort and use issue number ascending (the user is already filtering by category).
7. Apply limit.
8. Start processing immediately. Do not ask for confirmation.

## Step 2: Process Each Issue

For each issue in the queue:

1. Announce: `Starting issue #<number>: <title>`
2. Invoke the `/work-issue <number>` skill.
3. Track the outcome:
   - **Completed** — PR was opened successfully.
   - **Escalated** — needed user input, work paused.
   - **Skipped** — issue was unclear, out of scope, or required external changes.
4. After each issue, report status:
   ```
   ## Progress: N/M issues
   - ✅ #355 — PR #401 opened
   - ⏸️ #354 — escalated: needs design decision on separator byte
   - ⏭️ #353 — skipped: requires cross-repo change
   - ⏳ #352 — next up
   ```

## Step 3: Handle Escalations

When `/work-issue` escalates (design disagreement, product question, 3 review rounds exhausted):

1. Mark the issue as escalated with a clear reason.
2. **Move to the next issue immediately.** Do not stop to ask the user.
3. All escalations are reported in the final summary — the user will address them then.

## Step 4: Final Report

After all issues are processed (or limit reached), present a summary:

```
## Session Summary

### Completed (N)
- #355 — PR #401: fix host-indexer pagination
- #352 — PR #402: use chain-specific nonce type

### Escalated (N)
- #354 — waiting for: separator byte design decision
- #350 — waiting for: cross-session coupling approach

### Skipped (N)
- #353 — reason: requires host-substrate cross-repo changes

### Issues Filed
- #410 — deferred finding from #355 work
```

## Important Rules

- **Never ask for confirmation.** The user already told you what to work on — just do it.
- **One issue at a time.** Don't parallelize issue work — each needs a clean worktree.
- **Don't abandon an issue silently.** Every issue ends as completed, escalated, or skipped with a reason.
- **Respect the user's time.** Batch escalation questions when possible rather than interrupting for each one.
- **Clean up worktrees** after each issue.
- **Never mention AI tools in attribution.** Do not mention AI, assistants, automation, or generated-by wording in PRs, issue comments, commit messages, summaries posted back to GitHub, or any other user-facing attribution text.
