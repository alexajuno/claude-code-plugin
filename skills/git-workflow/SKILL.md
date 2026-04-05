---
name: git-workflow
description: >
  MUST use for ALL engineering tasks that involve code changes — any coding session,
  feature implementation, bug fix, refactor, or technical work that will result in
  commits or PRs. Also triggers on explicit git operations: commits, pushes, PR
  creation, PR updates (title/body/comments/review replies), branch cleanup, or
  slash commands /commit, /pr, /commit-push-pr, /clean-gone. Key overrides: no
  Co-Authored-By, no "Generated with Claude Code", Meta-style Test Plan section
  in PRs, Related PRs section with proper links. Uses REST API for PR updates to
  avoid GraphQL deprecation errors.
---

# Git Workflow

## Commit

1. Read current state: `git status`, `git diff HEAD`, `git log --oneline -10`
2. Stage relevant files (prefer specific files over `git add -A`)
3. Create a single commit with a concise message focused on the "why"
4. Execute all steps in a single message with parallel tool calls where possible

## Commit, Push & Create PR

1. Read current state: `git status`, `git diff HEAD`, `git branch --show-current`, `git log --oneline -10`
2. Create a new branch if on main/master
3. Stage and commit with an appropriate message
4. Push the branch with `-u` flag
5. Create PR using `gh pr create`

### PR Body Format

Use this format — no branding, no attribution:

```
gh pr create --title "<short title>" --assignee @me --body "$(cat <<'EOF'
## Motivation
<why this change is needed — what problem it solves or what goal it serves>

## Summary
<concise bullet points describing the changes>

## Test Plan
<how the changes were verified — commands run, manual steps taken, edge cases checked>
EOF
)"
```

If the user explicitly mentions a linked issue, replace the Motivation section with an issue reference:

```
Closes #<issue-number>

## Summary
...

## Test Plan
...
```

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

## Update PR

Use `gh api` REST endpoint instead of `gh pr edit` to avoid GraphQL deprecation errors (`gh pr edit --body` / `--title` may fail with "Projects (classic) is being deprecated...").

### Update Title / Body

```bash
PR_NUM=$(gh pr view --json number -q '.number')
REPO=$(gh repo view --json owner,name -q '"\(.owner.login)/\(.name)"')

# Update title
gh api repos/$REPO/pulls/$PR_NUM -X PATCH -f title="New Title"

# Update body (multi-line via file)
cat > /tmp/pr_body.txt << 'EOF'
## Motivation
...

## Summary
...

## Test Plan
...
EOF

gh api repos/$REPO/pulls/$PR_NUM -X PATCH -F "body=@/tmp/pr_body.txt"

# Update both at once
gh api repos/$REPO/pulls/$PR_NUM -X PATCH \
  -f title="New Title" \
  -F "body=@/tmp/pr_body.txt"
```

### Add PR Comment

```bash
gh pr comment $PR_NUM --body "$(cat <<'EOF'
## Review changes applied

Per @reviewer's review:
- Change 1
- Change 2
EOF
)"
```

### Reply to Review Comments

```bash
# List review comments (get IDs)
gh api /repos/$REPO/pulls/$PR_NUM/comments \
  --jq '.[] | {id, path, line: (.original_line // .line), body: .body}'

# Reply to a specific review comment
gh api /repos/$REPO/pulls/$PR_NUM/comments/{comment_id}/replies \
  -f body="Reply text"
```

### PR Update Guidelines

- **Body**: Current design/state — update when design changes, keep it clean
- **Comment**: Changelog entries — what changed and why (e.g., "Review changes applied")
- **Review reply**: Direct responses to inline code review comments

## Rules

- Never add Co-Authored-By or AI attribution to commits
- Never add "Generated with Claude Code" or similar to PR descriptions
- Always include a "Motivation" section in PRs explaining why the change is needed. If the user explicitly mentions a linked issue, replace Motivation with an issue reference (`Closes #N`) instead.
- Always include a "Test Plan" section in PRs (Meta style — concrete, reproducible verification steps)
- Commit message format:
  ```
  prefix: what

  why explain
  ```
  Common prefixes: feat, fix, refactor, docs, chore, test, style, ci
- Prefer staging specific files over `git add -A`
- Branch naming: `<prefix>/<short-description>` — use the same prefixes as commits (feat, fix, refactor, etc.). Never use `feature/`, always use `feat/`.
