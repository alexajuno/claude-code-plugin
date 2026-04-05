# Update PR — Title, Body, Comments

Use `gh api` REST endpoint instead of `gh pr edit` to avoid GraphQL deprecation errors.

## Get PR Info

```bash
# PR number for current branch
gh pr view --json number -q '.number'

# Owner/repo — use gh repo view, NOT gh pr view (headRepository.nameWithOwner can be empty)
REPO=$(gh repo view --json owner,name -q '"\(.owner.login)/\(.name)"')

# Full PR details
gh pr view --json number,title,body,url
```

## Update PR Title

```bash
gh api repos/$REPO/pulls/$PR_NUM -X PATCH -f title="New Title"
```

## Update PR Body

```bash
# Multi-line body via file
cat > /tmp/pr_body.txt << 'EOF'
## Motivation
...

## Summary
...

## Test Plan
...
EOF

gh api repos/$REPO/pulls/$PR_NUM -X PATCH -F "body=@/tmp/pr_body.txt"
```

## Update Both

```bash
gh api repos/$REPO/pulls/$PR_NUM -X PATCH \
  -f title="New Title" \
  -F "body=@/tmp/pr_body.txt"
```

## Add PR Comment

```bash
gh pr comment $PR_NUM --body "$(cat <<'EOF'
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
gh api /repos/$REPO/pulls/$PR_NUM/comments \
  --jq '.[] | {id, path, line: (.original_line // .line), body: .body}'

# Reply to a specific review comment
gh api /repos/$REPO/pulls/$PR_NUM/comments/{comment_id}/replies \
  -f body="Reply text"
```

## Why Not `gh pr edit`

`gh pr edit --body` or `gh pr edit --title` may fail with:
```
GraphQL: Projects (classic) is being deprecated...
```
The REST API bypasses this issue.

## Workflow Guidelines

- **Body**: Current design/state — update when design changes, keep it clean
- **Comment**: Changelog entries — what changed and why (e.g., "Review changes applied")
- **Review reply**: Direct responses to inline code review comments
