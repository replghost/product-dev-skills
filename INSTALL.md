# Installation

## Prerequisites

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed
- [GitHub CLI](https://cli.github.com/) (`gh`) installed and authenticated
- A GitHub repo to work with

## Install via Plugin Marketplace (Recommended)

```
/plugin marketplace add replghost/product-dev-skills
/plugin install product-dev-skills
```

To update later:

```
/plugin marketplace update product-dev-skills
```

## Install Manually (Alternative)

Copy skills and commands directly:

```bash
cp -r skills/* ~/.claude/skills/
cp commands/*.md ~/.claude/commands/
```

Or scope to a single project:

```bash
# From your project root
mkdir -p .claude/skills .claude/commands
cp -r /path/to/product-dev-skills/skills/* .claude/skills/
cp /path/to/product-dev-skills/commands/*.md .claude/commands/
```

## Configuration

The skills read project- and user-specific values from `git config` under the `product-dev-skills.*` namespace. The skills will prompt you to set anything they need on first use. To pre-configure:

The defaults below are the values that shipped with the skills before this change. Use them as-is to reproduce the original behavior, or replace each with what fits your project.

```bash
# Your GitHub username -- used to filter issues/PRs you authored.
git config --global product-dev-skills.github-author <your-username>

# Flags for `gh pr merge`. "--squash --admin" bypasses branch protection;
# drop "--admin" if you don't have admin access or want manual approval.
git config --global product-dev-skills.pr-merge-flags "--squash --admin"

# Paths to scan during /cleanup. Repeat with --add for multiple paths.
git config --global --add product-dev-skills.cleanup-paths "$HOME/dev/github/paritytech/host-sdk/target"
git config --global --add product-dev-skills.cleanup-paths "$HOME/dev/github/paritytech/host-sdk/.claude/worktrees"
git config --global --add product-dev-skills.cleanup-paths "$HOME/dev/github/paritytech/host-sdk-worktrees"
git config --global --add product-dev-skills.cleanup-paths "$HOME/.codex/worktrees"
git config --global --add product-dev-skills.cleanup-paths "$HOME/Library/Developer/Xcode/DerivedData"
git config --global --add product-dev-skills.cleanup-paths "$HOME/.gradle/caches"
```

`/bastion` embodies a demanding senior reviewer. You can create your own persona -- see below.

### Creating Your Own Reviewer Persona

To create a reviewer persona like `/bastion`:

1. Gather 10-20 real review comments from the person
2. Identify their patterns: what they always catch, what they reject, what they approve quickly
3. Write a command file following the `bastion.md` template
4. Save it to `~/.claude/commands/yourperson.md`

### Severity Tuning

Both `/work-issue` and `/review-pr` fix all severity levels by default. If you find that low-severity fixes introduce unnecessary churn:

- Edit the skills to skip Low-severity findings
- Or change them to file issues for Low findings instead of fixing inline

## Running the Full Stack

### Basic: Work a single issue

```bash
claude
# then: /work-issue 42
```

### Batch: Work all open issues

```bash
claude
# then: /work-issues
```

### Parallel teams: Split issues across terminals

```bash
# Terminal 1
claude -p "/work-issues --mod 3:0"

# Terminal 2
claude -p "/work-issues --mod 3:1"

# Terminal 3
claude -p "/work-issues --mod 3:2"
```

### Always-on PR reviewer

```bash
claude -p "/loop 10m /review-prs all"
```

### One-shot PR review

```bash
claude
# then: /review-pr 101
```
