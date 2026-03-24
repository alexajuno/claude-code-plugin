---
name: playwright-hygiene
description: Use when about to use any Playwright MCP tool (browser_navigate, browser_take_screenshot, browser_click, etc.) or when user mentions Playwright, browser testing, or taking screenshots of a web app. Ensures file hygiene, correct URLs, and dev server detection.
---

# Playwright Hygiene

Conventions for using the Playwright MCP server cleanly. Follow these rules whenever using `mcp__playwright__*` or `mcp__plugin_playwright_playwright__*` tools.

## Before First Navigation

**Always detect the dev server before guessing a URL.**

```bash
ss -tlnp 2>/dev/null | grep LISTEN || lsof -iTCP -sTCP:LISTEN 2>/dev/null
```

Cross-reference detected ports with the project:

| Indicator | Framework | Default Port |
|-----------|-----------|-------------|
| `package.json` with `vite` | Vite | 5173 |
| `composer.json` with `laravel` | Laravel | 8000 |

Use the detected URL. If no server is running, tell the user — do not navigate blindly.

## File Hygiene

**All Playwright artifacts go to `/tmp/playwright-session/`. Never save to the project directory.**

Before the first file-producing tool call, create the temp dir:

```bash
mkdir -p /tmp/playwright-session
```

### Screenshots

Always pass the `filename` parameter to `browser_take_screenshot`:

```
filename: /tmp/playwright-session/screenshot-01.png
```

Increment the number for each screenshot: `screenshot-01.png`, `screenshot-02.png`, etc.

### Console & Network Dumps

When saving console messages or network requests to file:

```
filename: /tmp/playwright-session/console.log
filename: /tmp/playwright-session/network.log
```

### Snapshots

When saving accessibility snapshots:

```
filename: /tmp/playwright-session/snapshot-01.md
```

## Cleanup

**When closing the browser or finishing a Playwright session:**

```bash
rm -rf /tmp/playwright-session
```

**Exception — keep on request:** If the user asks to "keep", "save", or "export" artifacts, ask where to copy them before cleaning up:

```bash
cp -r /tmp/playwright-session/ <user-specified-path>
rm -rf /tmp/playwright-session
```

## Quick Reference

| Rule | Details |
|------|---------|
| Artifacts dir | `/tmp/playwright-session/` |
| Screenshots | `screenshot-NN.png` with `filename` param |
| Console logs | `console.log` with `filename` param |
| Network logs | `network.log` with `filename` param |
| Snapshots | `snapshot-NN.md` with `filename` param |
| Detect server | `ss -tlnp` or `lsof` before navigating |
| Cleanup | `rm -rf /tmp/playwright-session` on close |
| Never | Save artifacts to project directory |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Screenshot without `filename` param | MCP saves to working dir — always pass `filename` |
| Guessing `localhost:3000` | Run port detection first |
| Forgetting cleanup after session | Always `rm -rf /tmp/playwright-session` when done |
| Navigating without checking if server is running | Detect first, tell user if nothing is running |
