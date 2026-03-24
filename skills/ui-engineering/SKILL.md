---
name: ui-engineering
description: >-
  Enforce component-driven development and design token management for frontend UI work.
  Use when building, modifying, or reviewing any frontend component, page, or view.
  Triggers on: creating UI components, styling elements, adding colors/spacing,
  building pages/views, working with forms/modals/alerts/tables, or any visual frontend task.
  Applies to any frontend framework (Vue, React, Svelte, etc.).
---

# UI Engineering

## Core Rules

1. **Never use raw HTML for interactive elements.** Search for existing base components first (`Base*.vue`, `ui/*.tsx`, etc.). Use them. Only create new base components when nothing fits.
2. **Never hardcode colors.** Every color must reference a CSS variable or design token. Search the project's global stylesheet for existing tokens before inventing new ones.
3. **Know the theme context.** Determine whether the target area uses light or dark theme. Use the correct token set — mixing dark tokens on light backgrounds makes text invisible.

## Before Writing Any Component

```
1. Glob for base/shared components:  **/{base,ui,shared,common}/**/*.{vue,tsx,jsx}
2. Read the global stylesheet:       **/app.scss, **/globals.css, **/variables.scss
3. Identify theme context:           Check the layout wrapper for data-bs-theme, class, or similar
4. List available design tokens:     Grep for --color- or --bs- in the stylesheet
```

Only after completing discovery, start implementation.

## Component Hierarchy

Build UI with this priority:

1. **Reuse** existing base component as-is
2. **Extend** existing base component with new prop/slot
3. **Create** new base component in `components/base/` (or project equivalent)
4. **Raw HTML** — only for static, non-interactive, one-off elements

When creating a new base component:
- Keep it generic (no business logic)
- Accept `variant`, `size`, or similar style props
- Style exclusively with design tokens
- Support slots for content flexibility

## Design Tokens

### Color Token Pattern

Organize status/semantic colors with consistent naming:

```
--color-{semantic}-bg        Background fill
--color-{semantic}-border    Border color
--color-{semantic}-text      Text/icon color
--color-{semantic}-solid     Full-strength color (buttons)
--color-{semantic}-solid-hover   Hover state for solid
```

Semantics: `success`, `warning`, `danger`, `info`, `primary`.

### When to Create New Tokens

- The color appears in 2+ components → extract to token
- The color represents a semantic meaning (status, action) → must be a token
- The color is decorative and used once → inline is acceptable

### Bootstrap Projects

Use `--bs-*` variables for structural styling (borders, backgrounds, text).
Use custom `--color-*` tokens for semantic/status colors.
Never mix them — `--bs-body-color` for body text, `--color-danger-text` for error text.

## Preventing Common UI Bugs

- **Layout shift on state change:** Never change `font-weight` on hover/active/focus. Use color, background, border, or shadow instead.
- **Theme mismatch:** Light-theme pages need light-compatible tokens. Verify by checking the actual background color of the target area.
- **Missing focus styles:** Interactive elements need visible focus indicators. Use `box-shadow` with `--color-primary-glow` or equivalent.

## Verification Checklist

Before delivering frontend code, verify:

- [ ] No raw `<button>`, `<input>`, `<select>` — base components used
- [ ] No hex/rgb/hsl literals in `<style>` — all via CSS variables
- [ ] Theme context matches token set (light tokens on light bg, dark on dark)
- [ ] Interactive states (hover, active, focus) don't cause layout shift
- [ ] New tokens added to global stylesheet, not component-scoped
