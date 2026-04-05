---
name: git-workflow
description: >
  MUST use for ALL engineering tasks that involve code changes — any coding session,
  feature implementation, bug fix, refactor, or technical work that will result in
  commits or PRs. Also triggers on explicit git operations: commits, pushes, PR
  creation, branch cleanup, or slash commands /commit, /pr, /commit-push-pr,
  /clean-gone. Key overrides: no Co-Authored-By, no "Generated with Claude Code",
  Meta-style Test Plan section in PRs, Related PRs section with proper links.
---

# Git Workflow

## Commit

1. Read current state: `git status`, `git diff HEAD`, `git log --oneline -10`
2. Stage relevant files (prefer specific files over `git add -A`)
3. Create a single commit with a concise message focused on the "why"
4. Execute all steps in a single message with parallel tool calls where possible
5. After a commit being pushed to a PR, fetch Gihub PR summary and align the content with the changes made by the commit. Take into account that we don't want to add new content because the change might be just local to the PR. We only add/update stuff related to the diff between targeted branch and the current branch, not changes inside current branch for the PR summary

## Commit, Push & Create PR

1. Read current state: `git status`, `git diff HEAD`, `git branch --show-current`, `git log --oneline -10`
2. Create a new branch if on main/master
3. Stage and commit with an appropriate message
4. Push the branch with `-u` flag
5. Create PR using `gh pr create`

### PR Body Format

Adapt the structure to the type of change. The first section sets the tone — pick the right one:

**For bug fixes and small changes** — use "Motivation" (what's broken, why it needs fixing):

```
gh pr create --title "<short title>" --assignee @me --body "$(cat <<'EOF'
## Motivation
<what's broken or missing on the base branch, and why it matters>

## Summary
<concise bullet points describing the changes>

## Test Plan
<how the changes were verified>
EOF
)"
```

**For features and large additions** — use "Context" (what this feature is, why it exists, where it fits):

```
gh pr create --title "<short title>" --assignee @me --body "$(cat <<'EOF'
## Context
<what this feature is, why the platform needs it, and how it fits into the bigger picture>

## Changes
<concise bullet points describing what was added/modified>

## Key Design Decisions
<non-obvious choices and their rationale — helps reviewers understand trade-offs>

## Test Plan
<how the changes were verified>
EOF
)"
```

For feature PRs with a design doc, include it as a collapsible `<details>` section at the end of the body.

**For linked issues** — replace the first section with an issue reference:

```
Closes #<issue-number>

## Summary / Changes
...

## Test Plan
...
```

### Writing the First Section

**Always write from the perspective of base branch → this branch.** Describe what's wrong or missing on the base branch (master/main), and what this PR changes. Never describe the internal evolution within the PR branch itself.

Good: "The Dockerfile is missing bash, which breaks sail shell. This PR adds it."
Bad: "We first added bash and git, then realized git was unnecessary, so we removed it."

### Test Plan Guidelines (Meta style)

The Test Plan answers: **"How do I know this works?"** Write it from the perspective of what was actually done during development, not hypothetical steps.

Include as applicable:
- **Automated tests:** which tests were added/modified, commands to run them (e.g., `npm test -- --filter=auth`)
- **Manual verification:** steps taken to verify behavior (e.g., "Verified login flow at /login with expired token")
- **Edge cases:** specific scenarios checked (e.g., "Tested with empty input, unicode characters, 10k rows")

Keep it concrete and actionable. A reviewer should be able to reproduce the verification.

### Related PRs Section

When there are related PRs (mentioned by the user, or discovered during work — e.g., a dependency, predecessor, or follow-up), add a **Related PRs** section after Test Plan:

```
## Related PRs
- [<repo>#<number> <short title>](<full GitHub URL>) — <role: e.g., "depends on", "predecessor", "follow-up", "splits off from", "replaces">
```

Example:
```
## Related PRs
- [zettelgraph#42 Add graph traversal API](https://github.com/user/zettelgraph/pull/42) — depends on (provides the API this PR consumes)
- [zettelgraph#38 Refactor node model](https://github.com/user/zettelgraph/pull/38) — predecessor (schema changes this PR builds upon)
```

Always use full markdown links (not bare URLs or `#N` shorthand) so they render properly in any context. Include a brief explanation of the relationship after the dash.

After PR creation, if the user mentioned a reviewer, immediately run `gh pr edit --add-reviewer <user>`. Don't just remind — do it.

Motivation (or issue link), Summary, and Test Plan are always included. Related PRs is included when applicable.

## Clean Gone Branches

Remove local branches whose remote has been deleted:

```bash
git branch -v | grep '\[gone\]' | sed 's/^[+* ]//' | awk '{print $1}' | while read branch; do
  worktree=$(git worktree list | grep "\\[$branch\\]" | awk '{print $1}')
  if [ ! -z "$worktree" ] && [ "$worktree" != "$(git rev-parse --show-toplevel)" ]; then
    git worktree remove --force "$worktree"
  fi
  git branch -D "$branch"
done
```

## Update PR (title, body, comments)

See `references/update-pr.md` for full patterns. Key point: use `gh api` REST endpoints, not `gh pr edit` (GraphQL deprecation errors).

## PR Review Comments (fetch & post inline)

See `references/pr-comments.md` for full patterns. Key points:
- Get repo with `gh repo view --json nameWithOwner` (not `gh pr view --json headRepository` — can return empty)
- Post inline comments via reviews API with `--input -` piped JSON (not `--field` — breaks JSON arrays)
- `line` must be a file-level line number within the diff hunk

## Rules

- PRs must open with context: "Motivation" for fixes/small changes, "Context" for features. If the user explicitly mentions a linked issue, replace with an issue reference (`Closes #N`) instead.
- Always include a "Test Plan" section in PRs (Meta style — concrete, reproducible verification steps)
- Commit message format:
  ```
  prefix: what

  why explain
  ```
  Common prefixes: feat, fix, refactor, docs, chore, test, style, ci
- Prefer staging specific files over `git add -A`
- Branch naming: `<prefix>/<short-description>` — use the same prefixes as commits (feat, fix, refactor, etc.). Never use `feature/`, always use `feat/`.
