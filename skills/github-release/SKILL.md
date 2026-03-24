---
name: github-release
description: Create a GitHub release for the current repo. Use when the user says "create a release", "release", "/github-release", or asks to tag/publish a new version. Pushes current branch and creates a release with auto-generated notes.
---

# Release

Create a GitHub release for the current git repository.

## Steps

1. Determine the repo root — run `git rev-parse --show-toplevel` from the current directory. If not in a git repo, ask the user which repo to use.
2. Fetch tags and determine the latest version:
   - Run `git fetch --tags` to ensure local tags are up to date
   - Run `gh release list --limit 10` to list recent releases
   - Identify the latest version **by semver** (highest `v<major>.<minor>.<patch>`), not by creation date
3. Analyze changes since the last tag:
   - Run `git log <last-tag>..HEAD --oneline` to list commits
   - Show the commit list to the user
   - Suggest a version bump based on commit content:
     - **Major** — if commits contain "BREAKING CHANGE", "breaking:", or remove/rename public APIs
     - **Minor** — if commits contain "feat:", "add", or introduce new functionality
     - **Patch** — if commits are mostly "fix:", "chore:", "refactor:", docs, or small tweaks
   - If the user specified a version, use it instead
   - Ask user to confirm or override the suggestion
4. `git push` the current branch
5. `gh release create <version> --generate-notes`
6. Return the release URL

## Rules

- Always operate on the current repo — never hardcode a repo path
- Title is always just the version tag (e.g. `v0.8.0`) — `--generate-notes` handles this
- Never use `--title` or `--notes` flags — always `--generate-notes` only
- Version format: `v<major>.<minor>.<patch>`
