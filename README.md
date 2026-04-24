# Product Dev Skills

*Hands-off software development with AI agent skills. Issue in, PR out, merged -- no human in the loop.*

This is the system behind 446 PRs merged and 344 issues closed in 30 days (~15 PRs/day, ~11 issues/day, ~10k lines of code/day).

## Features

- **Issue-to-PR pipeline** -- File a GitHub issue, walk away. An architect and tester agree on the plan, an implementer writes the code, reviewers catch problems, and a PR opens automatically.
- **Batch issue processing** -- Point it at your backlog. It prioritizes (security > bugs > enhancements), works issues one by one, and files follow-up issues for anything incomplete.
- **Automated PR review and merge** -- Reviews every open PR, fixes all findings (Critical through Low), runs CI, and squash-merges. Handles rebasing, conflict resolution, and dependency ordering across PRs.
- **Always-on mode** -- Loop the PR reviewer on a 10-minute interval. It triages new PRs, determines which can run in parallel, and processes them continuously.
- **Parallel teams** -- Split a batch of issues across multiple terminals. Each works an independent slice and files PRs simultaneously.
- **Demanding code reviewer persona** -- `/bastion` reviews your code the way a senior systems engineer would: root cause over workaround, correctness over convenience, minimal diffs, pointed "why?" questions.

## Quick Start

```
/plugin marketplace add replghost/product-dev-skills
/plugin install product-dev-skills
```

Then in any Claude Code session:

```bash
/work-issue 42           # Work a single issue end-to-end
/work-issues              # Batch through all open issues
/review-pr 101            # Review, fix, and merge a PR
/review-prs all           # Process all open PRs with smart scheduling
/bastion                  # Get a ruthless code review
/cleanup                  # Free disk space safely
```

See [INSTALL.md](INSTALL.md) for detailed setup and customization.

## Skills

| Skill | What it does |
|-------|-------------|
| **work-issue** | Full pipeline for one issue: plan, implement, review, fix, open PR |
| **work-issues** | Batch processor: loops through open issues, priority-sorted |
| **review-pr** | Review, fix all issues, and merge a single PR |
| **review-prs** | Multi-PR orchestrator with dependency analysis and parallel scheduling |
| **cleanup** | Safely remove stale worktrees and build caches |
| **bastion** | Demanding senior engineer code review persona |
| **wwbs** | "What Would Bastion Say?" -- alias for `/bastion` |
| **testing-patterns** | Testing patterns reference (auto-loads during test work) |

## How It Works

### Issue pipeline (`/work-issue`)

Two agents plan in parallel -- an architect designs the approach while a tester designs the test strategy. They must agree before any code gets written. An implementer then writes the code and tests, followed by parallel review from a tester and a reviewer. All findings get fixed in a loop (max 3 rounds). A coverage gate ensures nothing ships untested. The PR opens only after local checks pass.

### PR review (`/review-pr`)

Starts with a diff-stat overview, then targeted per-file review. Fixes every finding from Critical through Low. Runs cross-file consistency checks (duplicated constants, parallel implementations in different languages, stale generated bindings). Waits for CI, then squash-merges.

### Batch orchestration (`/review-prs`)

Triages PRs into ready, stale, superseded, and needs-author. Builds a dependency graph based on file overlap -- PRs touching different files run in parallel (batches of 4), PRs touching the same files form serial chains with CI waits between merges. CI fixes get priority over everything else.

### Architecture

```
+-------------------------------------------------------+
|                   Issue Definition                     |
|  Chat with AI -> File issue -> Delegate                |
|  (or: automated security/devex audits -> auto-file)    |
+----------------------------+--------------------------+
                             |
                             v
+-------------------------------------------------------+
|              /work-issue  (per issue)                  |
|                                                        |
|  Read issue -> Setup worktree ->                       |
|  Architect + Tester agree on plan ->                   |
|  Implementer writes code + tests ->                    |
|  Reviewer + Tester review in parallel ->               |
|  Fix loop (all severities, max 3 rounds) ->            |
|  Coverage gate -> CI checks -> Open PR                 |
+----------------------------+--------------------------+
                             |
              +--------------+--------------+
              v                             v
+------------------------+   +---------------------------+
|  /work-issues          |   |  Parallel teams           |
|  (batch processor)     |   |  Terminal 1: issues mod 3 |
|  Priority-sorted       |   |  Terminal 2: issues mod 3 |
|  Sequential per team   |   |  Terminal 3: issues mod 3 |
+-----------+------------+   +-------------+-------------+
            |                              |
            +---------------+--------------+
                            v
+-------------------------------------------------------+
|              /review-pr  (per PR)                      |
|                                                        |
|  Classify -> Setup worktree -> Rebase ->               |
|  Code review (diff-stat first, then targeted) ->       |
|  Fix ALL issues (Critical through Low) ->              |
|  Cross-file consistency checks ->                      |
|  CI checks -> Push -> Wait for CI -> Squash merge      |
+----------------------------+--------------------------+
                             |
                             v
+-------------------------------------------------------+
|              /review-prs  (batch orchestrator)         |
|                                                        |
|  Triage (stale/ready/superseded/needs-author) ->       |
|  Dependency analysis (file overlap) ->                 |
|  Priority classification (CI > bug > refactor > ...)   |
|  Independent PRs: parallel batches of 4                |
|  Conflicting PRs: serial chains with CI waits          |
|  Loop every 10 min for always-on reviewing             |
+-------------------------------------------------------+
```

## Design Decisions

**Two agents agree before work starts.** The architect and tester must align on the plan. If they disagree significantly, work stops and escalates. This prevents implementing something untestable or testing something architecturally wrong.

**Fix everything, not just critical issues.** The marginal cost of fixing a low-severity issue is near zero when the AI is already in context.

**Two layers of review.** Code gets reviewed during `/work-issue` (tester + reviewer) and again during `/review-pr` (full code review + cross-file consistency). The second pass regularly catches things the first missed.

**Automated follow-up issues.** If an agent can't fully satisfy requirements, it files a new GitHub issue. Nothing disappears into "follow-up."

## Usage Examples

### Always-on PR reviewer

```bash
claude -p "/loop 10m /review-prs all"
```

Scans for open PRs every 10 minutes, triages, determines parallelism, and processes them through the full review-fix-merge pipeline.

### Parallel issue teams

```bash
# Terminal 1
claude -p "/work-issues --mod 3:0"

# Terminal 2
claude -p "/work-issues --mod 3:1"

# Terminal 3
claude -p "/work-issues --mod 3:2"
```

### Automated issue generation

```bash
# Security audit -- file issues for every finding
claude -p "Analyze this repo as a security auditor. Focus on authentication, authorization, and data validation. File GitHub issues for every finding."

# Developer experience review
claude -p "Analyze this repo as a developer experience expert. Look at API ergonomics, documentation accuracy, and onboarding friction. File issues for improvements."
```

## Philosophy

1. **Do everything manually first.** Prompt, watch, see if it does what you expect.
2. **When you find a repeatable process, automate it.** Encode it as a skill.
3. **While waiting for AI, parallelize.** Run 5-10 terminal sessions. Start new work while others run.
4. **Continuously tune.** This is a living system. Try improvements manually first, then integrate.

Treat AI like a team member, not a tool. Give it its own job. Think like a manager optimizing parallel throughput, not an engineer reviewing every line. Prioritize end-to-end testing -- that's what enables hands-off work. And don't conserve tokens: if you can work at 3-10x speed, token conservation is a false economy.

## Practical Notes

- **Token limits are the real bottleneck.** The system can run 24/7, but you'll hit your allocation.
- **Disk space fills fast.** Multiple worktrees + compiled builds eat disk. Use `/cleanup` regularly.
- **Trust is earned incrementally.** Start by watching the output. Automate when comfortable.
- **Lower in the stack = more human oversight.** This works best for application-level code. Protocol code may want manual merge approval.

## License

[MIT](LICENSE)
