# PR Comments — Fetching & Posting

Reference for reading and posting inline review comments on GitHub PRs.

## Fetching Comments

```bash
# Get repo info — don't rely on headRepository.nameWithOwner (can be empty)
REPO=$(gh repo view --json nameWithOwner --jq '.nameWithOwner')
PR_NUM=$(gh pr view --json number --jq '.number')

# PR-level comments (general discussion)
gh api /repos/$REPO/issues/$PR_NUM/comments

# Inline review comments (on specific code lines)
gh api /repos/$REPO/pulls/$PR_NUM/comments
```

Key fields in review comments: `body`, `diff_hunk`, `path`, `line`, `side`, `in_reply_to_id`, `id`.

## Posting Inline Review Comments

Use the reviews API to post comments on specific lines of code.

### Finding the correct line number

The `line` parameter is the **line number in the file** (not the diff position). To comment on the right line:

1. Read the diff: `git diff master...HEAD` to see what changed
2. Identify the file-level line number you want to comment on
3. The line **must be part of the diff hunk** — you can only comment on added/modified lines (RIGHT side) or removed lines (LEFT side)

### API call format

**Must use `--input -` with a heredoc for the JSON body.** The `--field` flag treats JSON arrays as strings and causes 422 errors.

```bash
cat <<'PAYLOAD' | gh api repos/{owner}/{repo}/pulls/{number}/reviews -X POST --input -
{
  "commit_id": "<head commit SHA>",
  "event": "COMMENT",
  "body": "Review summary.",
  "comments": [
    {
      "path": "app/Services/SomeFile.php",
      "line": 42,
      "side": "RIGHT",
      "body": "Comment text here.\n\nMarkdown works:\n```php\n$code = 'example';\n```"
    }
  ]
}
PAYLOAD
```

### Getting the commit SHA

```bash
gh api repos/{owner}/{repo}/pulls/{number} --jq '.head.sha'
```

### Gotchas

- **`--field` vs `--input`**: `gh api --field 'comments=[...]'` serializes the array as a string, not JSON. Always use `--input -` with piped JSON for complex payloads.
- **Line must be in the diff**: Commenting on a line not part of the diff returns a 422 error. Verify with the diff first.
- **`side`**: Use `"RIGHT"` for new/modified lines, `"LEFT"` for removed lines.
- **Single quotes in body**: Escape carefully in heredocs. Use `<<'PAYLOAD'` (quoted delimiter) to prevent shell expansion.

## Replying to Existing Comments

```bash
gh api repos/{owner}/{repo}/pulls/{number}/comments/{comment_id}/replies \
  -X POST --field body="Reply text"
```
