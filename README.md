# a11y-audit

A Claude Code skill that runs a **WCAG accessibility audit** on any web application — automated axe scanning, keyboard navigation checks, screen reader testing, violation fixing, and audit log recording. Targets WCAG 2.2 AA by default; configurable per project or per run with `--wcag`.

Guides you through the whole process: Claude opens a browser, you navigate to the exact state you want tested (including logging in, opening dialogs, triggering error states), and then Claude runs the tests and reports findings. Multiple pages and states can be audited in a single session — findings accumulate into one report.

Works with React, Vue, Angular, Svelte, or plain HTML. Auto-detects framework, component library, dev server, and lint command.

---

## What It Does

1. **You navigate, Claude tests** — Claude opens a browser window and tells you what to do. You log in, open the right dialog, trigger the error state — whatever the test needs. When you're ready, Claude runs.
2. **Takes baseline screenshots** before touching code (saved outside the repo)
3. **Injects axe-core 4.10.3** via CDN and scans the current DOM against the selected WCAG standard (default: 2.2 AA — change with `--wcag`)
4. **Reports violations** grouped by impact: critical → serious → moderate → minor
5. **Checks keyboard navigation** — tab order, Escape-to-close, focus return, dialog behaviour
6. **NVDA screen reader check** *(optional, with `--nvda`)* — uses `nvda-testing-driver` (preferred) or NVDA Speech Viewer to capture spoken announcements. Windows only.
7. **VoiceOver screen reader check** *(optional, with `--voiceover`)* — works through the VoiceOver checklist with you using the Caption Panel for text output. macOS only.
8. **Structure, heading, and table check** — heading hierarchy, table markup, and landmark/region structure
9. **Form accessibility check** — labels, required fields, aria-invalid, aria-describedby, validation messages, field grouping
10. **Static analysis** *(optional, with `--static`)* — ESLint a11y plugin scan of source files; no browser required
11. **Loops across states** — after each state, Claude asks if there's another page or state to test; findings accumulate across the whole session
12. **Fixes violations** *(optional, with `--fix`)* — applies targeted, framework-appropriate fixes to all found violations
13. **Visual regression review** *(optional, with `--visual`)* — shows a side-by-side before/after comparison in the browser after each file is changed; you approve or reject before the change is kept
14. **Re-verifies** to zero violations at every checkpoint
15. **Takes after screenshots** for the audit record
16. **Updates the audit log** with numbered findings in a consistent format
17. **Runs lint** and confirms no new errors
18. **Exports a CSV** of findings *(optional, with `--report`)*

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

### 2. Set up Playwright MCP

The skill drives a real browser via the [Playwright MCP server](https://github.com/microsoft/playwright-mcp). It must be connected to your Claude Code session before you run the skill.

**Prerequisites:** Node.js 18 or later.

**Step 1 — Add Playwright MCP to Claude Code**

Open (or create) `~/.claude/settings.json` and add a `mcpServers` entry:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest"]
    }
  }
}
```

If the file already has other MCP servers, add the `"playwright"` key alongside them inside the existing `"mcpServers"` object.

**Step 2 — Install browsers**

Run once to download the Playwright browser binaries:

```bash
npx playwright install
```

To install only Chromium (smaller download):

```bash
npx playwright install chromium
```

**Step 3 — Restart Claude Code**

Close and reopen Claude Code so it picks up the new MCP server config.

**Step 4 — Verify the connection**

In a Claude Code session, ask:

```
What MCP tools do you have available?
```

You should see tools starting with `browser_` in the list (e.g. `browser_navigate`, `browser_click`, `browser_evaluate`). If they are missing, check that `~/.claude/settings.json` is valid JSON and restart again.

> **Windows note:** If `npx` is not found, make sure Node.js is on your `PATH`. You can verify with `node --version` in a new terminal.

### 3. Check your setup

Once the skill is installed and Claude Code is restarted, run a quick environment check:

```
/a11y-audit --check
```

This verifies that Playwright MCP is connected, detects your project's framework and lint command, checks what screen readers are available for your OS, and suggests the right command to start your first audit. No browser session is started and no audit runs — it's purely diagnostic.

Example output:

```
Accessibility audit — environment check

Playwright MCP
✓ Connected — browser tools available

Project
✓ Framework: React
✓ Lint command: npm run lint
⚠ No a11y-config.md found — will use auto-detection
⚠ No audit log found — will be created at docs/accessibility/AUDIT_LOG.md

Screen readers
✓ VoiceOver — built-in, always available (enable with Cmd+F5 before --voiceover tests)

Suggested command:
/a11y-audit "your page or feature" --voiceover --static

Or use the guided wizard:
/a11y-audit --wizard
```

### 4. Invoke the skill

```
/a11y-audit "sign-up form"
/a11y-audit "contact form" --fix
/a11y-audit "settings page" --report
```

Claude will open a browser and tell you exactly what to do from there.

---

## Invocation

```
/a11y-audit [label] [flags]
```

The label (optional) is a short name for what's being audited — used in the report and screenshots. If omitted, the short URL (e.g. `myapp.com/dashboard`) is used automatically once you navigate there.

```
/a11y-audit --check                       # verify environment and get a suggested command
/a11y-audit --wizard                      # guided setup — recommended for first-time users
/a11y-audit                               # URL will be used as the label automatically
/a11y-audit "sign-up form"               # label provided, all tests
/a11y-audit "settings page" --code       # axe scan only
/a11y-audit "contact form" --fix         # all tests + auto-fix
/a11y-audit "navigation menu" --visual   # all tests + visual review per fix
/a11y-audit "profile page" --static --code # source analysis + axe
/a11y-audit "main nav" --report          # all tests + CSV export
/a11y-audit "footer" --static            # source analysis only (no browser needed)
/a11y-audit "sign-up form" --voiceover  # VoiceOver screen reader check (macOS)
/a11y-audit "sign-up form" --nvda       # NVDA screen reader check (Windows)
/a11y-audit "dashboard" --wcag 2.1-AA   # target WCAG 2.1 AA instead of default 2.2
```

---

## Options

### Setup

| Flag | What it does |
|---|---|
| `--check` | Environment check — verifies Playwright MCP is connected, detects your framework and lint command, checks screen reader availability for your OS, and suggests the right command to start your first audit. Run once after installing. No audit is started. |
| `--wizard` | Interactive guided setup — Claude asks what you want to audit, explains each test in plain language, covers fix and report options, and walks through what will happen before anything runs. No flags to remember. Ideal for first-time users or unfamiliar audits. |

### Scope

| Flag | What it does |
|---|---|
| `--wcag <target>` | WCAG version and level to target. Default: `2.2-AA`. Examples: `--wcag 2.1-AA`, `--wcag 2.0-AA`, `--wcag 2.2-A`, `--wcag 2.2-AAA`. Drives the axe tag set and is recorded in the audit log and CSV report. |

### No additional requirements

| Flag | What it does |
|---|---|
| `--static` | ESLint a11y plugin scan of source files — no browser, no dev server. Works standalone. Catches click-without-keyboard-handler, invalid ARIA in source, and issues in components not reached during live testing. Installs the right plugin for your framework if needed. |
| `--report` | Writes a CSV of all findings to the screenshots folder. Works with any combination of other flags. |
| `--fresh-report` | Like `--report`, but deletes any existing CSV for this label before writing — starts a clean report rather than appending. |

### Requires Playwright MCP

| Flag | What it does |
|---|---|
| *(no test flags)* | All four browser tests — axe, keyboard, structure, forms — in report-only mode |
| `--code` | Axe automated scan only |
| `--keyboard` | Keyboard navigation check only |
| `--structure` | Heading hierarchy, table markup, and landmark structure only |
| `--forms` | Form labels, required fields, validation messages, and field grouping only |

### Requires Playwright MCP — modifiable source only

| Flag | What it does |
|---|---|
| `--fix` | After reporting, automatically fix all found violations. Use only when the source files are in your working directory. |
| `--visual` | Like `--fix`, but opens a side-by-side before/after comparison in the browser after each file is changed — you approve or reject before the change is kept. Implies `--fix`. |

### Requires a screen reader

| Flag | What it does |
|---|---|
| `--nvda` | NVDA screen reader check — always opt-in, never runs by default. Uses `nvda-testing-driver` if installed, otherwise NVDA Speech Viewer. **Windows only.** Requires NVDA installed. |
| `--voiceover` | VoiceOver screen reader check — always opt-in, never runs by default. Walks through `references/voiceover-checklist.md` with you using the VoiceOver Caption Panel for text output. Best results with Safari. **macOS only.** VoiceOver is built-in — no install needed. |

---

## What to Expect

When you run the skill, here's what a typical session looks like:

**Step 1 — Claude detects your project and asks where to save files:**

> **Where would you like screenshots and temporary files saved?**
>
> This includes before/after screenshots, visual comparison pages, and the CSV report (if using `--report`). These must be **outside your project folder** — file watchers inside the project trigger hot-reload every time a file is written.
>
> Default: `C:\Users\you\Pictures\a11y-screenshots\`
>
> Type a different path, or reply `default` to use the path shown above.

You confirm the path (or type a new one), and Claude creates the folder if it doesn't exist.

**Step 2 — Claude summarises what it found and what will run:**

> **Accessibility audit ready**
>
> Auditing: *sign-up form*
> Detected: React · Lint: `npm run lint`
>
> Tests queued: axe · keyboard · structure · forms
> Mode: report only
>
> Screenshots will be saved to: `C:\Users\you\Pictures\a11y-screenshots\`
> Findings will be added to: `docs/accessibility/AUDIT_LOG.md`

**Step 3 — Claude opens a browser and tells you what to do:**

> **Your turn — navigate to the state you want tested**
>
> The browser is open. Please:
> 1. Type the URL of your app into the browser address bar and press Enter
> 2. Log in if required
> 3. Navigate to the specific page, flow, or feature you want to audit
> 4. If you want to test a dialog or modal, open it now
> 5. Get the browser into exactly the state you want audited
>
> When everything looks right, say **ready**.

You log in, navigate to the login page (or open the dialog, or trigger the error state — whatever you need). Then you say **ready**.

**Step 4 — Claude confirms what it sees and starts testing:**

> Got it — I can see the login form with email and password fields.
>
> I'll now run: axe · keyboard · structure · forms
>
> First up: axe automated scan. This will take a moment — I'll update you as each test completes.

Each test announces when it starts and reports its result when it finishes:

> Axe scan complete — 2 violations (1 serious, 1 moderate)
>
> Keyboard check complete — 1 issue found
>
> Structure check complete — all clear
>
> Form check complete — 1 issue found

**Step 5 — Summary and loop:**

> **Results for: login form**
>
> | Test | Issues |
> |---|---|
> | Axe | 2 |
> | Keyboard | 1 |
> | Structure | 0 |
> | Forms | 1 |
>
> ---
> **Is there another page or state you'd like to add to this audit?**
>
> Navigate there in the browser and say **ready**, or say **done** if you've finished.

You can add more states — for example, navigate to the error state (submit the form empty) and say **ready** again. Claude will test that state too and add to the same session report.

When you're done, say **done** and Claude proceeds to record everything in the audit log (and fix violations if you passed `--fix`).

---

## First Time?

If you've just installed the skill, run the environment check first:

```
/a11y-audit --check
```

It takes a few seconds, tells you what's available, and prints the exact command to start your audit. No browser session is opened and no code is touched.

Once you know your setup works, either use the suggested command directly or try the guided wizard:

```
/a11y-audit --wizard
```

## Wizard Mode

If you're not sure which flags to use, or you're running your first audit, the wizard walks you through everything:

```
/a11y-audit --wizard
```

Claude will walk you through everything in plain language before any tests run. Here's what the conversation looks like:

**Claude asks what you're auditing:**
> First — **what are we auditing today?**
>
> Give it a short name that describes the page, flow, or feature. Examples: `sign-up form`, `settings page`, `contact form`, `item selection dialog`
>
> *Type `skip` if you don't have a label yet — I'll derive one from the URL when you navigate there.*

**Claude explains each test and asks which ones you want:**
> Here are the tests available. I'll briefly explain each one:
>
> **1. Axe automated scan** — Injects axe-core into the live page and checks for WCAG 2.2 AA violations automatically. Catches colour contrast failures, missing labels, invalid ARIA, nested interactive elements, and more.
>
> **2. Keyboard navigation** — I'll tab through the page and check that every interactive element is reachable, focus indicators are visible, tab order matches the visual layout, and dialogs close on Escape.
>
> **3. Structure** — Checks heading levels, table markup, and landmark regions.
>
> **4. Forms** — Checks field labels, required fields, validation errors, and error message announcements.
>
> **5. NVDA screen reader** — Drives the browser with NVDA running alongside. Requires NVDA. **Windows only.**
>
> **6. VoiceOver screen reader** — Same coverage using Apple VoiceOver. No install needed — built into macOS. Best with Safari. **macOS only.**
>
> **7. Static analysis** — Scans your source files with an ESLint a11y plugin. No browser needed.
>
> Which would you like? Options:
> - **All browser tests** (1–4, recommended — works everywhere)
> - **All + screen reader** (1–4 + NVDA on Windows, or 1–4 + VoiceOver on macOS)
> - **Everything** (1–4 + screen reader + static)
> - **Custom** — tell me which numbers

**Claude asks about fixing and reporting:**
> After I report the findings, what would you like to do with them?
>
> **Report only** — Document everything but don't touch your code.
> **Fix automatically** — Work through each affected file after reporting.
> **Fix with visual review** — Same as fix, but show a before/after comparison for each file so you can approve or reject.

**Claude explains the browser navigation model:**
> Here's what's going to happen next:
>
> 1. I'll scan your project to detect your framework and configuration
> 2. I'll ask where to save screenshots and temp files (outside your project folder)
> 3. I'll open a browser window — it will start blank, that's expected
> 4. I'll ask you to navigate to the page or state you want tested — log in, open the right dialog, trigger the error state — then say **ready**
> 5. I'll run the selected tests and report findings
> 6. After each state, I'll ask if there's another to add — findings accumulate into one session report
> 7. At the end, I'll record everything in the audit log

**Claude confirms and starts:**
> Ready to go? Say **yes** to start, or ask any questions first.

Once you confirm, the audit runs exactly the same as if you'd passed the equivalent flags directly.

---

## How Visual Review Works

When `--visual` is active, after each file is saved the skill:

1. Waits for the dev server to hot-reload
2. Takes a screenshot of the affected UI state
3. Generates a side-by-side before/after comparison page and opens it in the Playwright browser
4. Asks you: **does this look acceptable?**
   - **Yes** — keeps the change and moves on
   - **No** — reverts the file to its original state, marks the finding as deferred, and moves on

This lets you catch any unintended visual side-effects from a fix before they land.

---

## Requirements

- **Claude Code** with the Playwright MCP server connected (for browser tests)
- **NVDA** installed (Windows only) for `--nvda` — either `nvda-testing-driver` for automation or NVDA Speech Viewer for manual capture
- **VoiceOver** (macOS only) for `--voiceover` — built-in, no install needed; enable with Cmd+F5 and set up the Caption Panel before running
- **Source files in the working directory** for `--fix` and `--visual`

No dev server configuration required — you navigate the browser yourself, so you control how to reach any page or state.

---

## Project-Specific Configuration

Create an `a11y-config.md` file in your project root to override auto-detection:

```markdown
# a11y-config

screenshots: C:\Users\you\Pictures\a11y-screenshots\
audit-log: docs/accessibility/AUDIT_LOG.md
component-library: [your component library name]
lint: yarn lint:web
```

See `references/project-config.md` for the full field reference and template.

---

## WCAG Coverage (default: 2.2 AA)

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

### Manual (keyboard + screen reader checks)

| Criterion | Check |
|---|---|
| 2.4.11 Focus Appearance | 2px outline, 3:1 contrast, not obscured by sticky headers |
| 2.5.7 Dragging Movements | Single-pointer alternative for any drag operation |
| 2.5.8 Target Size (Minimum) | All targets ≥ 24×24 CSS pixels |
| 3.3.7 Redundant Entry | Fields pre-populated from earlier steps |
| 3.3.8 Accessible Authentication | No cognitive-only auth (CAPTCHAs, transcription) |

---

## Coverage and Limitations

### What this skill covers

- **~57 WCAG 2.2 AA success criteria** — axe-core automates roughly 40% of detectable issues; the manual keyboard, screen reader, structure, and forms checks cover most of the rest
- **Automated:** colour contrast, missing labels, invalid ARIA, nested interactive controls, missing alt text, focus order, landmark regions, heading hierarchy, table markup
- **Manual:** focus appearance (2.4.11), target size (2.5.8), dragging alternatives (2.5.7), redundant entry (3.3.7), accessible authentication (3.3.8), NVDA/VoiceOver announcement quality

### What this skill does NOT cover

| Gap | Why |
|---|---|
| **ARIA design pattern compliance** | Complex widgets (data grid, tree, combobox, date picker) have keyboard interaction patterns defined by the ARIA Authoring Practices Guide. Axe checks ARIA attributes but not full keyboard behaviour. Requires specialist review. |
| **Cognitive accessibility** | WCAG 2.2 touches on this (3.3.7, 3.3.8) but broader cognitive load, plain language, and reading level are outside scope. |
| **Mobile / touch** | The audit runs in a desktop browser. Touch target sizes, gesture alternatives, and mobile screen reader behaviour (TalkBack, VoiceOver on iOS) are not tested. |
| **Multi-browser variance** | Tests run in Chromium via Playwright. Some ARIA support differences exist in Safari and Firefox. Supplementary testing in other browsers is recommended for public-facing work. |
| **Performance as an accessibility dimension** | Slow load times and layout shifts affect users with cognitive and motor disabilities, but are outside WCAG scope. |
| **PDF and document accessibility** | Only web content in the browser is tested. |

### Keeping axe up to date

The axe CDN URL in Step 5 is pinned to version **4.10.3**. When a new version releases, update the URL in `SKILL.md`:

```
https://cdnjs.cloudflare.com/ajax/libs/axe-core/4.10.3/axe.min.js
                                                   ^^^^^^^
```

Check [cdnjs.com/libraries/axe-core](https://cdnjs.com/libraries/axe-core) for the latest version.

### Browser

Playwright MCP defaults to **Chromium**. Axe results, keyboard behaviour, and ARIA support are representative of Chrome/Edge. For public-facing products, supplementary testing in Safari (especially if targeting macOS/iOS users) and Firefox is recommended.

---

## Reference Files

| File | Contents |
|---|---|
| `references/fix-patterns.md` | WCAG rule → fix patterns, organised by rule ID with generic HTML and framework-agnostic React examples |
| `references/keyboard-checks.md` | Keyboard navigation check protocol with reporting format |
| `references/nvda-checklist.md` | NVDA testing via `nvda-testing-driver` and Speech Viewer — expected announcement formats, checklist template |
| `references/voiceover-checklist.md` | VoiceOver testing on macOS via the Caption Panel — key commands, expected announcement formats, checklist template |
| `references/structure-checks.md` | Heading hierarchy, table markup, and landmark structure check protocol with reporting format |
| `references/form-checks.md` | Form label, required field, validation message, and field grouping check protocol with reporting format |
| `references/audit-log-format.md` | How to create and update audit logs, including finding entry format and CSV column spec |
| `references/project-config.md` | `a11y-config.md` template for project-specific overrides |

---

## Hard Rules

These apply in every project and cannot be overridden by `a11y-config.md`:

- **Never navigate the browser without the user's instruction** — the user drives navigation; Claude drives testing
- **Never submit forms or trigger destructive actions** without warning the user first
- **Never save screenshots or temp files inside the project folder** — file watchers trigger hot-reload on every write
- **Never save the same file more than once per task** — batch all edits into one save per file
- **Never commit without explicit user instruction**
- **Re-inject axe after every `browser_navigate`** — it does not persist between page navigations

---

## Fix Pattern Library

The `references/fix-patterns.md` file covers the most common violations by axe rule ID, with generic HTML and framework-agnostic React examples that apply to any component library:

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

## Collaboration

Questions, ideas, or want to work together on accessibility tooling? Get in touch:

- **GitHub:** [github.com/kylane](https://github.com/kylane)
- **LinkedIn:** [linkedin.com/in/kylane](https://www.linkedin.com/in/kylane/)

---

## Background

This skill was extracted from an accessibility audit workflow developed on a production web application. The workflow used Playwright MCP + axe-core CDN injection to audit a multi-step wizard and associated dialogs, finding and fixing multiple WCAG violations in a single session.

Key design decisions:

- **User-driven navigation** — Claude opens the browser; the user logs in and navigates to the exact state they want tested. This handles auth, deep states, open dialogs, and multi-step flows naturally without any URL config or auth scripting
- **Axe CDN injection** (no npm install required) works for ad-hoc audits in any project without modifying `package.json`
- **Multi-state scanning** catches violations that only appear when dialogs or overlays are open
- **Stack-agnostic fix patterns** — examples use generic HTML and framework-neutral React so the guidance applies regardless of component library
- **`a11y-config.md`** keeps project-specific overrides out of the generic skill
- **Screenshots outside the repo** — critical constraint; files inside the project folder pollute git and trigger file-watcher reloads

---

## License

MIT
