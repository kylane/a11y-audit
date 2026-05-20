# a11y-audit

A Claude Code skill that runs a full **WCAG 2.2 AA** accessibility audit on any web application — automated axe scanning, keyboard navigation checks, NVDA screen reader testing, violation fixing, and audit log recording.

Works with React, Vue, Angular, or plain HTML. Auto-detects framework, component library, dev server, and lint command.

---

## What It Does

1. **Auto-detects** framework, component library, dev server URL, lint command, and existing audit log
2. **Takes baseline screenshots** before touching code (saved outside the repo)
3. **Injects axe-core 4.10.3** via CDN at every distinct UI state (page at rest, dialogs, wizard steps, error states)
4. **Reports violations** grouped by impact: critical → serious → moderate → minor
5. **Checks keyboard navigation** — tab order, Escape-to-close, focus return, dialog behaviour
6. **NVDA screen reader check** — uses `nvda-testing-driver` (preferred) or NVDA Speech Viewer mode to capture spoken announcements and assess them against expected formats
7. **Fixes violations** using framework-appropriate patterns from the included reference library
8. **Re-verifies** to zero violations at every checkpoint
9. **Takes after screenshots** for visual regression review
10. **Updates the audit log** with numbered findings in a consistent format
11. **Runs lint** and confirms no new errors

---

## Quick Start

### 1. Install

```bash
# Clone to your skills directory
git clone https://github.com/lanedigital/a11y-audit ~/.agents/skills/a11y-audit

# Symlink into Claude Code's skills directory
# macOS / Linux
ln -s ~/.agents/skills/a11y-audit ~/.claude/skills/a11y-audit

# Windows (run as Administrator, or use Developer Mode)
New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\.claude\skills\a11y-audit" -Target "$env:USERPROFILE\.agents\skills\a11y-audit"
```

### 2. Make sure Playwright MCP is connected

The skill uses Playwright MCP tools (`browser_navigate`, `browser_evaluate`, `browser_click`, etc.) to drive the browser and inject axe. Confirm Playwright MCP is listed in your Claude Code session before running the skill.

### 3. Start your dev server

The skill does not start the server — it must already be running.

### 4. Invoke the skill

```
/a11y-audit create record wizard
/a11y-audit login and registration flow
/a11y-audit grants selection dialog
/a11y-audit          # audits whatever is currently open in the browser
```

---

## Requirements

- **Claude Code** with the Playwright MCP server connected
- **Dev server running** before invocation (the skill will not start it)
- **NVDA** installed (Windows only) for screen reader checks — either `nvda-testing-driver` for automation or NVDA Speech Viewer for manual capture

---

## Project-Specific Configuration

Create an `a11y-config.md` file in your project root to override auto-detection:

```markdown
# a11y-config

url: http://localhost:4200
auth: Log in as admin@example.com / password123 before auditing
screenshots: C:\Users\you\Pictures\a11y-screenshots\
audit-log: docs/accessibility/AUDIT_LOG.md
component-library: Chakra UI
lint: yarn lint:web
```

See `references/project-config.md` for the full field reference and template.

---

## WCAG 2.2 AA Coverage

### Automated (axe-core)

| Criterion | Rule IDs |
|---|---|
| 1.1.1 Non-text Content | `image-alt` |
| 1.3.1 Info and Relationships | `list`, `label`, `scope-attr-valid` |
| 1.4.3 Contrast (Minimum) | `color-contrast` |
| 2.1.1 Keyboard | `scrollable-region-focusable` |
| 2.1.2 No Keyboard Trap | `dialog-name` + missing `onClose` |
| 2.4.3 Focus Order | `tabindex` |
| 4.1.2 Name, Role, Value | `nested-interactive`, `dialog-name`, `button-name`, `aria-hidden-focus` |

### Manual (keyboard + NVDA checks)

| Criterion | Check |
|---|---|
| 2.4.11 Focus Appearance | 2px outline, 3:1 contrast, not obscured by sticky headers |
| 2.5.7 Dragging Movements | Single-pointer alternative for any drag operation |
| 2.5.8 Target Size (Minimum) | All targets ≥ 24×24 CSS pixels |
| 3.3.7 Redundant Entry | Fields pre-populated from earlier steps |
| 3.3.8 Accessible Authentication | No cognitive-only auth (CAPTCHAs, transcription) |

---

## Reference Files

| File | Contents |
|---|---|
| `references/fix-patterns.md` | WCAG rule → fix patterns, organised by rule ID and component library (MUI, Radix, generic HTML) |
| `references/keyboard-checks.md` | Keyboard navigation check protocol with reporting format |
| `references/nvda-checklist.md` | NVDA testing via `nvda-testing-driver` and Speech Viewer — expected announcement formats, checklist template |
| `references/audit-log-format.md` | How to create and update audit logs, including finding entry format |
| `references/project-config.md` | `a11y-config.md` template for project-specific overrides |

---

## Hard Rules

These apply in every project and cannot be overridden by `a11y-config.md`:

- **Never guess route paths** — always navigate via the app's own UI (sidebar, nav menu, links)
- **Never save screenshots or temp files inside the project folder** — file watchers trigger hot-reload on every write
- **Never save the same file more than once per task** — batch all edits into one save per file
- **Never commit without explicit user instruction**
- **Re-inject axe after every `browser_navigate`** — it does not persist between page navigations

---

## Fix Pattern Library

The `references/fix-patterns.md` file covers the most common violations by axe rule ID, with framework-specific examples for MUI, Radix UI, and generic HTML:

- `nested-interactive` — checkbox/input inside a button or link
- `list` — invalid direct children of `<ul>` / `<ol>`
- `dialog-name` — dialog has no accessible name
- Missing `onClose` — dialog not keyboard-dismissible (Escape)
- `label` — missing or excessively long field labels
- `scrollable-region-focusable` — scrollable container not keyboard-reachable
- `color-contrast` — insufficient colour contrast ratio
- `image-alt` — decorative or meaningful image missing alt text
- `button-name` — icon button with no label
- `aria-hidden-focus` — focusable element hidden from accessibility tree
- `region` — content not within a landmark
- `tabindex` — positive tabindex breaks natural tab order
- Plus WCAG 2.2 new criteria: 2.4.11, 2.5.7, 2.5.8, 3.3.7, 3.3.8

---

## Contributing

To add fix patterns for a new component library, add a section to `references/fix-patterns.md` following the existing format (rule ID → library → wrong → correct code).

To extend the NVDA checklist for a new widget type, add a section to `references/nvda-checklist.md`.

Pull requests welcome.

---

## Background

This skill was extracted from an accessibility audit workflow developed on a production React/MUI application. The workflow used Playwright MCP + axe-core CDN injection to audit a multi-step record creation wizard and associated dialogs, finding and fixing 7 WCAG violations across 9 files in a single session.

Key design decisions:

- **Axe CDN injection** (no npm install required) works for ad-hoc audits in any project without modifying `package.json`
- **Multi-state scanning** catches violations that only appear when dialogs or overlays are open
- **Framework-specific patterns** — the fix for `nested-interactive` in MUI differs from plain HTML
- **`a11y-config.md`** keeps project-specific auth and navigation knowledge out of the generic skill
- **Screenshots outside the repo** — critical constraint; files inside the project folder pollute git and trigger file-watcher reloads

---

## License

MIT
