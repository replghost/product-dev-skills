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

## Customization

These skills were built for a specific stack and workflow. They work out of the box, but you'll get better results by adapting them to your project. Here's where to look:

### Things you should customize

1. **Author filter in work-issues**: The skill filters issues by `--author replghost`. Change this to your GitHub username so it only picks up issues you filed.
2. **Auto-merge behavior in review-pr**: Uses `gh pr merge --squash --admin`. Remove `--admin` if you don't have admin access, or remove the merge step entirely if you want manual merge approval.
3. **Local checks in work-issue and review-pr**: These run `cargo fmt`, `cargo clippy`, `cargo test`, `pnpm tsc`, and changeset checks. Replace with whatever your project uses (e.g., `npm test`, `go vet`, `pytest`).
4. **PR review checklist in review-pr**: The checklist looks for UniFFI binding staleness, Swift/Android parity, and duplicated constants. Replace these with your own cross-cutting concerns (e.g., migration safety, API versioning, i18n).
5. **Cleanup paths in cleanup**: The disk paths are specific to one machine's layout. Update the `du -sh` paths to match your own build directories and worktree locations.
6. **Bastion persona**: `/bastion` embodies a demanding senior reviewer. You can create your own persona -- see below.

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
