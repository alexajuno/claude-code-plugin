---
name: update-pr
description: Full PR workflow — update title/body, add comments, reply to review comments. Use when user asks to edit PR title, update PR description, add a PR comment, reply to review feedback, or when `gh pr edit` fails with GraphQL deprecation errors.
---

# Update PR

Use `gh api` REST endpoint instead of `gh pr edit` to avoid GraphQL deprecation errors.

## Update PR Title

```bash
gh api repos/{owner}/{repo}/pulls/{pr_number} -X PATCH -f title="New Title"
```

## Update PR Body

```bash
# Simple body
gh api repos/{owner}/{repo}/pulls/{pr_number} -X PATCH -f body="New description"

# Multi-line body via file
cat > /tmp/pr_body.txt << 'EOF'
## Summary
- Feature 1
- Feature 2

## Test plan
- [x] Tested locally
EOF

gh api repos/{owner}/{repo}/pulls/{pr_number} -X PATCH -F "body=@/tmp/pr_body.txt"
```

## Update Both

```bash
gh api repos/{owner}/{repo}/pulls/{pr_number} -X PATCH \
  -f title="New Title" \
  -F "body=@/tmp/pr_body.txt"
```

## Get PR Info

```bash
# PR number for current branch
gh pr view --json number -q '.number'

# Owner/repo
gh repo view --json owner,name -q '"\(.owner.login)/\(.name)"'

# Full PR details
gh pr view --json number,title,body,url
```

## Common Pattern

```bash
PR_NUM=$(gh pr view --json number -q '.number')
REPO=$(gh repo view --json owner,name -q '"\(.owner.login)/\(.name)"')

gh api repos/$REPO/pulls/$PR_NUM -X PATCH -f title="Updated title"
```

## Why Not `gh pr edit`

Avoid `gh pr edit --body` or `gh pr edit --title` - they may fail with:
```
GraphQL: Projects (classic) is being deprecated...
```

The REST API bypasses this issue.

## Add PR Comment

```bash
# Add a comment to a PR
gh pr comment {pr_number} --body "comment text"

# Multi-line via heredoc
gh pr comment {pr_number} --body "$(cat <<'EOF'
## Review changes applied

Per @reviewer's review:
- Change 1
- Change 2
EOF
)"
```

## Reply to Review Comments

```bash
# List review comments (get IDs)
gh api /repos/{owner}/{repo}/pulls/{pr_number}/comments \
  --jq '.[] | {id, path, line: (.original_line // .line), body: .body}'

# Reply to a specific review comment
gh api /repos/{owner}/{repo}/pulls/{pr_number}/comments/{comment_id}/replies \
  -f body="Reply text"
```

## Workflow Guidelines

- **Body**: Current design/state — update when design changes, keep it clean
- **Comment**: Changelog entries — what changed and why (e.g., "Review changes applied")
- **Review reply**: Direct responses to inline code review comments
