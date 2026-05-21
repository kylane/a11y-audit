---
name: a11y-audit
description: Run a WCAG accessibility audit (configurable version and level — default 2.2 AA) using static analysis + axe-core + keyboard + NVDA + VoiceOver + structure + forms on any web app. Guides the user through each step — open a browser, navigate to the state you want tested, say ready. Works with any framework or stack.
triggers:
  - "run accessibility audit"
  - "a11y audit"
  - "wcag audit"
  - "accessibility check"
  - "axe audit"
  - "check accessibility"
  - "audit for accessibility"
  - "wcag check"
  - "a11y check"
  - "screen reader test"
  - "check for accessibility issues"
  - "accessibility audit"
  - "test accessibility"
---

# Accessibility Audit Skill

A guided WCAG 2.2 AA accessibility audit. The user opens a browser, navigates to the exact page or state they want tested (including logging in, opening dialogs, triggering specific states), and says ready. Claude then runs the selected tests and reports findings. Multiple pages and states can be audited in one session — findings accumulate into a single report.

Works with any web framework or tech stack. Auto-detects project structure where possible; asks when it cannot.

---

## Skill Invocation

```
/a11y-audit [label] [flags]
```

The label (optional) is a short name for what's being audited — used in the report and screenshots. If omitted, the short URL (e.g. `myapp.com/dashboard`) is used automatically once you navigate there.

**Examples:**
```
/a11y-audit --check                            # verify environment and get a suggested command
/a11y-audit --wizard                           # guided setup — Claude walks you through everything
/a11y-audit                                    # URL will be used as the label automatically
/a11y-audit "sign-up form"                     # label provided
/a11y-audit "settings page" --code             # axe scan only
/a11y-audit "contact form" --fix               # all tests + auto-fix
/a11y-audit "navigation menu" --visual         # all tests + visual review per fix
/a11y-audit "profile page" --static --code     # source analysis + axe
/a11y-audit "main nav" --report                # all tests + CSV export
```

---

## Flags

### Setup flags

| Flag | What it does |
|---|---|
| `--install` | Installation guide — asks where to install the skill (current project or global), then outputs the exact commands to run. Exits after printing commands — no audit is started. |
| `--check` | Environment check — verifies Playwright MCP is connected, detects your project setup, and suggests the right command to start your first audit. Run this after installing the skill. |
| `--wizard` | Interactive setup — Claude asks what you want to audit, explains each test, and walks you through the whole process in plain language before anything runs. Ideal for first-time users or when you're not sure which flags to use. |

### Scope flags

| Flag | What it does |
|---|---|
| `--url <url>` | Open the browser directly to this URL and run tests immediately — no manual navigation required. If no label is provided, it is derived from the URL automatically. Subsequent states in the session still use manual navigation. |
| `--wcag <target>` | WCAG version and level to target. Default: `2.2-AA`. Accepted values: `2.0-A`, `2.0-AA`, `2.0-AAA`, `2.1-A`, `2.1-AA`, `2.1-AAA`, `2.2-A`, `2.2-AA`, `2.2-AAA`. Affects the axe scan tag set and is recorded in the audit log and CSV. |

### Test flags

| Flag | What it runs | Requires |
|---|---|---|
| *(none)* | All four browser tests: axe + keyboard + structure + forms | Playwright MCP |
| `--static` | ESLint a11y plugin scan of source files — no browser needed | None |
| `--code` | Axe automated scan only | Playwright MCP |
| `--keyboard` | Keyboard navigation check only | Playwright MCP |
| `--nvda` | NVDA screen reader check — always opt-in, never a default. Windows only. | Playwright MCP + NVDA |
| `--voiceover` | VoiceOver screen reader check — always opt-in, never a default. macOS only. | Playwright MCP + VoiceOver |
| `--structure` | Heading, table, and landmark structure check only | Playwright MCP |
| `--forms` | Form label, required field, and validation check only | Playwright MCP |

### Action flags

| Flag | Description |
|---|---|
| `--fix` | After reporting, automatically fix all found violations. Source code must be accessible in the working directory. |
| `--visual` | Like `--fix`, with a per-file before/after comparison in Playwright — approve or reject each change. Implies `--fix`. |

### Output flags

| Flag | Description |
|---|---|
| `--report` | Generate or update a CSV report in the screenshots folder with all findings from this session. |
| `--fresh-report` | Like `--report`, but deletes any existing CSV for this label before writing — starts a clean report rather than appending. |
| `--plan` | Generate a Markdown remediation plan — a shareable document with prioritised findings, recommended fixes, and next steps. Saved alongside the CSV in the screenshots folder. |

Test flags can be combined freely. `--static` works without a browser and can be combined with browser tests. `--fix` and `--visual` assume source code is in the working directory — do not use them when auditing a site you cannot modify.

---

## Implementation

### Phase 1: Setup

#### Step 0I: Installation *(runs when `--install` is passed — exits after printing commands, does not start an audit)*

When the user passes `--install`, do not start an audit. Instead, guide them through installation.

Note the current working directory. Then ask:

> **Where would you like to install the a11y-audit skill?**
>
> - **This project** (default) — `.claude/skills/a11y-audit/` inside your current project
>   Available only in this directory.
>
> - **Global** — `~/.claude/skills/a11y-audit/` (macOS/Linux) or `$env:USERPROFILE\.claude\skills\a11y-audit` (Windows)
>   Available across all your Claude Code projects.
>
> - **Custom path** — type a directory and I'll use it.

Wait for the response. Then output the appropriate commands:

**This project (macOS/Linux):**
```bash
git clone https://github.com/lanedigital/a11y-audit .claude/skills/a11y-audit
```

**This project (Windows):**
```powershell
git clone https://github.com/lanedigital/a11y-audit .claude\skills\a11y-audit
```

**Global (macOS/Linux):**
```bash
git clone https://github.com/lanedigital/a11y-audit ~/.claude/skills/a11y-audit
```

**Global (Windows):**
```powershell
git clone https://github.com/lanedigital/a11y-audit "$env:USERPROFILE\.claude\skills\a11y-audit"
```

**Custom:** substitute the user-provided path for the destination.

Then add:

> **Next steps after running that command:**
>
> 1. Add Playwright MCP to `~/.claude/settings.json` if you haven't already — see the README for the config block
> 2. Run `npx playwright install` to download browser binaries
> 3. Restart Claude Code
> 4. Run `/a11y-audit --check` to verify the setup
>
> Then start your first audit with `/a11y-audit --wizard`

After printing the instructions, stop. Do not proceed to Step 0C, Step 0W, or Step 0.

---

#### Step 0C: Environment check *(runs when `--check` is passed — exits after reporting, does not start an audit)*

When the user passes `--check`, run the following checks and print a structured report. Do not start an audit.

---

**Check 1 — Playwright MCP**

Attempt to open the browser:
```
browser_navigate → about:blank
```

- Success → ✓ Playwright MCP connected
- Tool not available / error → ✗ Playwright MCP not connected

If connected, also detect the OS:
```js
browser_evaluate → navigator.platform
```
- Starts with `Win` → Windows
- Starts with `Mac` → macOS
- Other → Linux / unknown

---

**Check 2 — Project**

- Read `package.json` if present; detect framework, component library, and lint command using the same logic as Step 1
- Check whether `a11y-config.md` exists in the project root
- Check for an existing audit log (same search order as Step 1)
- Check whether `nvda-testing-driver` is in `package.json` dependencies

---

**Check 3 — Screen readers**

Based on detected OS:
- **Windows:** Note that NVDA must be installed separately. Check if `nvda-testing-driver` is in `package.json`. Report either "nvda-testing-driver available (automated capture)" or "NVDA manual — Speech Viewer mode will be used"
- **macOS:** VoiceOver is built-in — always available. Note that it must be enabled manually before running `--voiceover`.
- **Linux / unknown:** Neither NVDA nor VoiceOver available — screen reader tests not supported.

---

**Output the report:**

> **Accessibility audit — environment check**
>
> **Playwright MCP**
> ✓ Connected — browser tools available *(or ✗ Not connected — browser tests unavailable)*
>
> **Project** *(if package.json found)*
> ✓ Framework: *[React / Vue / Angular / Svelte / Unknown]*
> ✓ Component library: *[name / not detected]*
> ✓ Lint command: *[command / none found]*
> ✓ a11y-config.md: *[found / not found — will use auto-detection]*
> ✓ Audit log: *[path if found / will be created at docs/accessibility/AUDIT_LOG.md]*
>
> **Screen readers**
> *[Windows:]*
> ✓ NVDA — *[nvda-testing-driver available / Speech Viewer mode — NVDA must be running before `--nvda` tests]*
> *[macOS:]*
> ✓ VoiceOver — built-in, always available *(enable with Cmd+F5 before running `--voiceover` tests)*
> *[Linux / unknown:]*
> ⚠ Screen reader tests not supported on this OS
>
> ---
> **Suggested command:**
>
> *[Build the suggestion from what's available — see logic below]*
>
> Or use the guided setup wizard:
> `/a11y-audit --wizard`

**Suggestion logic:**

| Playwright | OS | Suggestion |
|---|---|---|
| ✓ Connected | Windows | `/a11y-audit "[label]"` *(all 4 browser tests)* · add `--nvda` *(NVDA must be running — Speech Viewer mode works even without `nvda-testing-driver`)* · add `--static` if a JS framework was detected |
| ✓ Connected | macOS | `/a11y-audit "[label]"` *(all 4 browser tests)* · add `--voiceover` · add `--static` if a JS framework was detected |
| ✓ Connected | Linux / unknown | `/a11y-audit "[label]"` *(all 4 browser tests)* · add `--static` if a JS framework was detected |
| ✗ Not connected | Any | `/a11y-audit "[label]" --static` *(source analysis only — no browser needed)* · explain how to set up Playwright MCP |

If Playwright is not connected, also print:

> **To enable browser tests**, add Playwright MCP to `~/.claude/settings.json`:
> ```json
> {
>   "mcpServers": {
>     "playwright": {
>       "command": "npx",
>       "args": ["@playwright/mcp@latest"]
>     }
>   }
> }
> ```
> Then run `npx playwright install` and restart Claude Code. See the README for full setup instructions.

After printing the report, stop. Do not proceed to Step 0W or Step 0.

---

#### Step 0W: Wizard setup *(runs when `--wizard` is passed — skip to Step 0 otherwise)*

When the user passes `--wizard`, do not parse flags or start any tests. Instead, run the following guided conversation to collect everything needed. At the end of Step 0W, all variables will be set exactly as they would be by Step 0, and the audit proceeds from Step 1 as normal.

Check for `a11y-config.md` first — if present, note that it exists and that its settings will apply. Then begin:

---

**0W-1 — Welcome**

> **Welcome to the accessibility audit wizard.**
>
> I'll ask you a few quick questions to set up your audit, then walk you through the whole process before anything runs. This should take about two minutes.
>
> If you already know the flags you want, you can skip this and run `/a11y-audit [label] [flags]` directly — but this wizard is a good way to get familiar with what's available.

---

**0W-2 — What are we auditing?**

> First — **what are we auditing today?**
>
> Give it a short name that describes the page, flow, or feature. This label will be used in the audit log and screenshots.
>
> Examples: `sign-up form`, `settings page`, `contact form`, `navigation menu`, `item selection dialog`
>
> *Type `skip` if you don't have a label yet — I'll derive one from the URL when you navigate there.*

Wait for the user's answer. If they provide a label, set `LABEL` from it. If they say "skip", send a blank message, or provide nothing, set `LABEL = PENDING`.

---

**0W-3 — Which tests?**

Use `AskUserQuestion` with 2 questions:

**Question: "Which tests would you like to run?"**
- Header: `Tests`
- Option 1 — label: `All browser tests`, description: `Axe · keyboard · structure · forms — recommended, works everywhere`
- Option 2 — label: `All + screen reader`, description: `All browser tests plus NVDA (Windows) or VoiceOver (macOS)`
- Option 3 — label: `All + static`, description: `All browser tests plus ESLint source file scan`
- Option 4 — label: `Everything`, description: `All browser tests + screen reader + static analysis`
- *(User selects a custom set via the automatic "Other" option — interpret generously e.g. "just axe" → RUN_CODE only)*

Set `RUN_CODE`, `RUN_KEYBOARD`, `RUN_NVDA`, `RUN_VOICEOVER`, `RUN_STRUCTURE`, `RUN_FORMS`, `RUN_STATIC` from the answer.

**Question: "Which WCAG standard are we targeting?"**
- Header: `WCAG target`
- Option 1 — label: `2.2 AA`, description: `Default — current published standard, recommended for most projects`
- Option 2 — label: `2.1 AA`, description: `Widely required by accessibility legislation`
- Option 3 — label: `2.0 AA`, description: `Older baseline, broadly supported`
- *(User types a custom target e.g. `2.2-AAA` via "Other")*

Set `WCAG_TARGET` from the answer, defaulting to `2.2-AA` if nothing is provided.

---

**0W-4 — Fix mode?**

Use `AskUserQuestion` with 1 question:

**Question: "What would you like to do with the findings after reporting?"**
- Header: `After audit`
- Option 1 — label: `Report only`, description: `Document findings in the audit log — don't touch code. Good for external sites or when you want to review before fixing.`
- Option 2 — label: `Fix automatically`, description: `Apply fixes to each affected file after reporting. Requires source files in the working directory.`
- Option 3 — label: `Fix with visual review`, description: `Fix + before/after screenshot comparison in browser — approve or reject each change.`

Set `FIX_VIOLATIONS` = true for Fix automatically or Fix with visual review. Set `VISUAL_REVIEW` = true for Fix with visual review.

---

**0W-5 — Where to save files?**

Determine the OS-appropriate default path:
- Windows: `C:\Users\<username>\Pictures\a11y-screenshots\`
- macOS/Linux: `~/Pictures/a11y-screenshots/`

Use `AskUserQuestion` with 1 question:

**Question: "Where would you like screenshots and temporary files saved?"**
- Header: `Screenshots`
- Note before options: "Must be outside your project folder — file watchers trigger hot-reload on every write."
- Option 1 — label: `Default`, description: the detected default path
- *(User types a custom path via the automatic "Other" option)*

Set `SCREENSHOT_PATH` to the typed path, or to the platform default if "Default" is chosen. Create the folder if it doesn't already exist.

---

**0W-6 + 0W-6b — Outputs?**

Use `AskUserQuestion` with 2 questions:

**Question: "Would you like a CSV report at the end?"**
- Header: `CSV report`
- Option 1 — label: `Yes`, description: `Append to existing report for this label`
- Option 2 — label: `Yes, fresh`, description: `Delete any existing report and start clean`
- Option 3 — label: `No`

Set `EXPORT_REPORT` = true for Yes or Yes, fresh. Set `FRESH_REPORT` = true for Yes, fresh.

**Question: "Would you like a shareable remediation plan?"**
- Header: `Remediation plan`
- Option 1 — label: `Yes`, description: `Markdown doc with prioritised findings and fix guidance — ready to share with a team or client`
- Option 2 — label: `No`

Set `EXPORT_PLAN` = true for Yes.

---

**0W-7 — Explain the process**

> Here's what's going to happen next:
>
> **1. I'll scan your project** to detect your framework, component library, and lint configuration.
>
> **2. I'll open a browser window.** It will start blank — that's expected.
>
> **3. I'll ask you to navigate to the page or state you want tested.**
> You control the browser — type the URL, log in, click through to the right page, open the dialog or modal you want tested, trigger the error state if needed. Get it into exactly the right state, then say **ready**.
> *You do not need a running dev server configured anywhere — just open the browser and navigate as you normally would.*
>
> **4. I'll run the selected tests** on whatever is currently in the browser. I'll announce what I'm doing before each test and report the result when it's done.
>
> **5. After each state, I'll ask if there's another page or state to add** — so you can cover multiple flows in one session. Findings accumulate into a single report.
>
> **6. At the end**, I'll record all findings in the audit log*[, fix violations]*[, generate a CSV report]*[, and generate a remediation plan]*.
>
> Ready to go? Say **yes** to start, or ask any questions first.

Wait for the user to confirm. If they have questions, answer them, then ask again. Once confirmed, set:

- `STATES_TESTED` = empty list
- `ALL_FINDINGS` = empty list

And proceed to Step 1.

---

#### Step 0: Parse flags and load config *(runs when `--wizard` was NOT passed)*

Check for `a11y-config.md` in the project root. If found, read it for project-specific overrides (screenshot path, audit log path, component library notes, lint command).

Set the following variables from the invocation flags:

- `LABEL` = the label argument if provided; otherwise `PENDING` — the short URL will be used as the label once the browser is navigated in Step 4
- `AUTO_URL` = the `--url` argument if provided; otherwise empty
- `FULL_URL` = not yet set — will be captured from the browser in Step 4
- `WCAG_TARGET` = the `--wcag` argument if provided; otherwise the `wcag.wcag` value from `a11y-config.md` if present; otherwise `2.2-AA`
- `WCAG_TAGS` = the axe tag set derived from `WCAG_TARGET` — see table below
- `RUN_STATIC` = true if `--static` was passed
- `ANY_TEST_FLAG` = true if any of `--static`, `--code`, `--keyboard`, `--nvda`, `--voiceover`, `--structure`, `--forms` was passed
- `RUN_CODE` = true if `--code` was explicitly passed, OR if `ANY_TEST_FLAG` is false
- `RUN_KEYBOARD` = true if `--keyboard` was explicitly passed, OR if `ANY_TEST_FLAG` is false
- `RUN_NVDA` = true **only** if `--nvda` was explicitly passed — never a default
- `RUN_VOICEOVER` = true **only** if `--voiceover` was explicitly passed — never a default
- `RUN_STRUCTURE` = true if `--structure` was explicitly passed, OR if `ANY_TEST_FLAG` is false
- `RUN_FORMS` = true if `--forms` was explicitly passed, OR if `ANY_TEST_FLAG` is false
- `FIX_VIOLATIONS` = true if `--fix` or `--visual` was passed
- `VISUAL_REVIEW` = true if `--visual` was passed
- `EXPORT_REPORT` = true if `--report` or `--fresh-report` was passed
- `FRESH_REPORT` = true if `--fresh-report` was passed
- `EXPORT_PLAN` = true if `--plan` was passed
- `SCREENSHOT_PATH` = from `a11y-config.md` if specified; otherwise determined and confirmed in Step 1
- `STATES_TESTED` = empty list — will accumulate each state tested this session
- `ALL_FINDINGS` = empty list — will accumulate all findings across all states

**WCAG_TAGS lookup** — derive from `WCAG_TARGET`:

| WCAG_TARGET | axe tags |
|---|---|
| `2.0-A` | `['wcag2a']` |
| `2.0-AA` | `['wcag2a', 'wcag2aa']` |
| `2.0-AAA` | `['wcag2a', 'wcag2aa', 'wcag2aaa']` |
| `2.1-A` | `['wcag2a', 'wcag21a']` |
| `2.1-AA` | `['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa']` |
| `2.1-AAA` | `['wcag2a', 'wcag2aa', 'wcag2aaa', 'wcag21a', 'wcag21aa', 'wcag21aaa']` |
| `2.2-A` | `['wcag2a', 'wcag21a']` *(no new A-level criteria in 2.2)* |
| `2.2-AA` *(default)* | `['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa', 'wcag22aa']` |
| `2.2-AAA` | `['wcag2a', 'wcag2aa', 'wcag2aaa', 'wcag21a', 'wcag21aa', 'wcag21aaa', 'wcag22aa']` |

Accept case-insensitive level input (`aa` = `AA`). If the value is unrecognised, default to `2.2-AA` and warn the user.

**Checkpoint resume:** If `LABEL` is `PENDING`, skip this check — no checkpoint can exist without a resolved label. Otherwise, after setting `LABEL` and `SCREENSHOT_PATH`, check for a checkpoint file at `<SCREENSHOT_PATH>/a11y-<label-slug>-checkpoint.json`. If it exists, tell the user:

> I found a saved checkpoint for "*[LABEL]*" from *[checkpoint date]*. It has *N* states tested and *N* findings recorded.
>
> Would you like to **resume** from where you left off, or **start fresh**?

If they say resume: load `STATES_TESTED` and `ALL_FINDINGS` from the checkpoint, then skip directly to Step 3.
If they say fresh: delete the checkpoint file and continue from Step 1 as normal.

#### Step 1: Detect project context

Read `package.json` if present. If not present, note "no package.json found" and continue — the skill works with any stack.

**Framework** (from `package.json` dependencies, if available):
- `react` / `react-dom` → React
- `vue` → Vue
- `@angular/core` → Angular
- `svelte` → Svelte
- No match or no `package.json` → Unknown / non-JS stack

**Component library** (from `package.json` dependencies, if available):
- `@mui/material` → MUI · `@chakra-ui/react` → Chakra UI · `@radix-ui/*` → Radix UI
- `@headlessui/react` / `@headlessui/vue` → Headless UI · `primereact` / `primevue` → PrimeReact/Vue
- `antd` → Ant Design · No match → Unknown / custom

**Package manager** (from lockfiles):
- `yarn.lock` → yarn · `pnpm-lock.yaml` → pnpm · `package-lock.json` → npm · None → ask if needed

**Lint command** (from `package.json` scripts, if available):
- `lint`, `lint:web`, `lint:app`, `lint:frontend` → use matching script
- Not found → skip lint step

**Screenshot output path:**

If `SCREENSHOT_PATH` is already set (from `a11y-config.md` or the wizard in Step 0W-5), skip this step.

Otherwise, determine the OS-appropriate default:
- Windows: `C:\Users\<username>\Pictures\a11y-screenshots\`
- macOS/Linux: `~/Pictures/a11y-screenshots/`

**Setup questions — ask all at once using `AskUserQuestion`:**

Build a single `AskUserQuestion` call containing whichever of these three questions are still unset. Skip any whose flag was already provided.

**Question: "Where would you like screenshots saved?"**
- Header: `Screenshots`
- Option 1 — label: `Default`, description: the detected default path (e.g. `C:\Users\ky\Pictures\a11y-screenshots\`)
- *(User types a custom path via the automatic "Other" option)*

Set `SCREENSHOT_PATH` to the typed path, or to the platform default if "Default" is chosen. Create the folder if it doesn't already exist.

**Question: "Would you like a CSV report at the end?"** *(skip if `EXPORT_REPORT` already set)*
- Header: `CSV report`
- Option 1 — label: `Yes`, description: `Append to existing report for this label`
- Option 2 — label: `Yes, fresh`, description: `Delete any existing report and start clean`
- Option 3 — label: `No`

Set `EXPORT_REPORT` = true for Yes or Yes, fresh. Set `FRESH_REPORT` = true for Yes, fresh.

**Question: "Would you like a shareable remediation plan?"** *(skip if `EXPORT_PLAN` already set)*
- Header: `Remediation plan`
- Option 1 — label: `Yes`, description: `Markdown doc with prioritised findings and fix guidance`
- Option 2 — label: `No`

Set `EXPORT_PLAN` = true for Yes.

**Audit log:**
- Check in order: `docs/ux/AUDIT_LOG.md` → `docs/AUDIT_LOG.md` → `AUDIT_LOG.md` → `.accessibility/audit-log.md`
- If none found: create `docs/accessibility/AUDIT_LOG.md` using the template in `references/audit-log-format.md`

**Tell the user what was detected:**

> **Accessibility audit ready**
>
> Auditing: *[LABEL — or "label will be set from page URL" if LABEL is PENDING]*
> Standard: *[WCAG_TARGET e.g. "WCAG 2.2 AA"]*
> Detected: *[framework + component library, or "unknown stack"]* · Lint: *[command or "none found"]*
>
> Tests queued: *[list active tests e.g. "axe · keyboard · structure · forms"]*
> Mode: *[e.g. "report only" or "fix" or "fix with visual review"]*
> Outputs: *[list active outputs — e.g. "audit log · CSV · remediation plan"]*
>
> Screenshots will be saved to: *[path]*
> Findings will be added to: *[audit log path]*

#### Step 2: Static analysis *(runs when RUN_STATIC is true)*

**Skip if `RUN_STATIC` is false.**

Tell the user:
> Running static analysis — scanning your source files with the ESLint a11y plugin. No browser needed for this step.

Identify the correct plugin for the detected framework:

| Framework | Plugin | Ruleset |
|---|---|---|
| React | `eslint-plugin-jsx-a11y` | `plugin:jsx-a11y/recommended` |
| Vue | `eslint-plugin-vuejs-accessibility` | `plugin:vuejs-accessibility/recommended` |
| Angular | `@angular-eslint/eslint-plugin` | `plugin:@angular-eslint/recommended` |
| Svelte | `eslint-plugin-svelte` | `plugin:svelte/recommended` |
| Unknown | Note that static analysis is not available without a detected JS framework; skip |

Check `package.json` and the ESLint config for the plugin. Three states:

- **Already installed and configured** → run the existing lint command, filter for a11y rule violations
- **Installed but not configured** → ask: *"The a11y plugin is installed but not in your ESLint config. Add it permanently, or run an inline scan for this audit only?"*
- **Not installed** → ask: *"[plugin] isn't installed. Want me to install it and add it to your ESLint config?"* If no, skip and continue.

Report findings grouped by rule ID then file. Note any violations likely to be caught by axe too — these are cross-validated findings.

Add all violations to `ALL_FINDINGS` with `test_type: static` and `url: ""` (static findings are not tied to a specific browser URL).

Tell the user the result:
> Static analysis complete — *[N violations found / no violations]* *[(brief breakdown if any)]*

---

### Phase 2: Browser-guided test loop

This phase repeats for each page or state the user wants to test. Each iteration adds findings to `ALL_FINDINGS` and a record to `STATES_TESTED`.

#### Step 3: Open browser and guide the user

**If `AUTO_URL` is set and `STATES_TESTED` is empty (first state, URL provided):**

Navigate directly to the URL:
```
browser_navigate → AUTO_URL
```

Tell the user:
> Opened *[AUTO_URL]* — running tests now.

Proceed immediately to Step 4 without waiting for the user.

---

**If `AUTO_URL` is empty and `STATES_TESTED` is empty (first state, no URL provided):**

Navigate to `about:blank` to open the browser:
```
browser_navigate → about:blank
```

Tell the user exactly what to do:

> **Your turn — navigate to the state you want tested**
>
> The browser is open. Please:
> 1. Type the URL of your app into the browser address bar and press Enter
> 2. Log in if required
> 3. Navigate to the specific page, flow, or feature you want to audit
> 4. If you want to test a dialog or modal, open it now
> 5. If you want to test an error state, trigger it now (e.g. submit a form with missing fields)
> 6. Get the browser into exactly the state you want audited
>
> When everything looks right, say **ready**.

Wait. Do not proceed until the user confirms.

---

**Subsequent states** (when `STATES_TESTED` is not empty): the browser is already open. Do not navigate — the user will go to the next state themselves.

Tell the user:

> **Is there another page or state you'd like to add?**
>
> Navigate there in the browser and say **ready**, or say **done** to finish.

Wait. Do not proceed until the user confirms.

#### Step 4: Confirm state and take baseline screenshot

When the user says ready, take a snapshot to confirm the current state:

```
browser_snapshot
```

**Capture the URL.** Evaluate:
```js
browser_evaluate → window.location.href
```

Set `FULL_URL` = the full URL returned (e.g. `https://www.myapp.com/dashboard`).

**If `LABEL` is `PENDING`**, derive the short URL label using these rules:
- Strip protocol (`https://`, `http://`)
- Strip `www.` prefix
- Strip query string (everything from `?` onward)
- Strip URL fragment (everything from `#` onward)
- Strip trailing `/` if it's the root path only (no other path segments)
- Keep port if non-standard (not 80 for http, not 443 for https)

Examples:
- `https://www.myapp.com/dashboard` → `myapp.com/dashboard`
- `http://localhost:3000/settings` → `localhost:3000/settings`
- `https://staging.myapp.com/` → `staging.myapp.com`
- `https://app.example.com/users?page=2#section` → `app.example.com/users`

Set `LABEL` = the derived short URL. From this point forward, `LABEL` is a real string and `PENDING` is cleared.

Store `FULL_URL` in the per-state record so it is written to the checkpoint and CSV per finding.

Tell the user what you can see and confirm the plan:

> Got it — I can see *[description of the current page/state e.g. "the sign-up form" or "the settings page with the preferences panel open"]*
>
> *[If LABEL was PENDING: "I'll use "*[short URL]*" as the label for this state."]*
>
> I'll now run: *[list of active tests]*
>
> First up: *[name of first test]*. This will take a moment — I'll update you as each test completes.

Take a baseline screenshot:
```
<output-path>/a11y-<label-slug>-<state-slug>-<n>-before.png
```

Where `<n>` is the iteration number (1 for the first state, 2 for the second, etc.), and `<label-slug>` is the label with spaces replaced by hyphens and special characters (`.`, `/`, `:`) replaced by hyphens.

#### Step 5: Axe scan *(runs when RUN_CODE is true)*

**Skip if `RUN_CODE` is false.**

Tell the user:
> Running axe automated scan — checking for *[WCAG_TARGET]* violations in the current DOM...

Inject and run axe. **Re-inject after every `browser_navigate`** — it does not persist between navigations.

Use `WCAG_TAGS` (derived in Step 0 from `WCAG_TARGET`) as the tag set for the scan.

```js
const script = document.createElement('script');
script.src = 'https://cdnjs.cloudflare.com/ajax/libs/axe-core/4.10.3/axe.min.js';
document.head.appendChild(script);
await new Promise(resolve => script.onload = resolve);
const results = await axe.run(document, {
  runOnly: { type: 'tag', values: WCAG_TAGS }  // set from WCAG_TARGET in Step 0
});
return {
  violations: results.violations.map(v => ({
    id: v.id, impact: v.impact, description: v.description,
    nodes: v.nodes.length, failureSummary: v.nodes[0]?.failureSummary, html: v.nodes[0]?.html
  })),
  count: results.violations.length
};
```

Scan at each distinct UI state visible from the current browser position. Report violations grouped by impact (critical → serious → moderate → minor). Cross-reference any findings also caught by static analysis.

Tell the user the result:
> Axe scan complete — *[N violations / no violations]* *[(e.g. "2 critical, 1 moderate")]*

#### Step 6: Keyboard navigation check *(runs when RUN_KEYBOARD is true)*

**Skip if `RUN_KEYBOARD` is false.**

Tell the user:
> Running keyboard navigation check — tabbing through the page to check focus order, indicators, and traps...

Load `references/keyboard-checks.md`. Verify:
- Every interactive element reachable by Tab
- Focus indicator visible on each focused element
- Tab order matches visual/semantic reading order
- No focus traps
- Dialogs: focus enters on open, Escape closes, focus returns to trigger on close

Use `browser_press_key`: `Tab`, `Shift+Tab`, `Enter`, `Space`, `Escape`, `ArrowUp`, `ArrowDown`.

Tell the user the result:
> Keyboard check complete — *[N issues / all clear]* *[(brief summary if issues found)]*

#### Step 7: Screen reader check *(runs when RUN_NVDA or RUN_VOICEOVER is true)*

**Skip if both `RUN_NVDA` and `RUN_VOICEOVER` are false.**

---

**NVDA (Windows) — runs when RUN_NVDA is true**

Before starting, tell the user what's needed:

Tell the user: "For the NVDA check, please make sure NVDA is running (system tray) and Speech Viewer is open: NVDA menu → Tools → Speech Viewer. Or `nvda-testing-driver` in `package.json` enables automated capture."

Use `AskUserQuestion` with 1 question:

**Question: "Is NVDA ready?"**
- Header: `NVDA`
- Option 1 — label: `Ready`, description: `NVDA is running and Speech Viewer is open`
- Option 2 — label: `Skip`, description: `Continue without the NVDA check`

If Skip, note it and move on.

Load `references/nvda-checklist.md`. Use `nvda-testing-driver` if installed (check `package.json`), otherwise use Speech Viewer approach — after each key interaction, ask the user to share the Speech Viewer output.

Tell the user the result:
> NVDA check complete — *[N issues / all clear]*
>
> *[If using Speech Viewer: "Thanks for your help with those announcements."]*

---

**VoiceOver (macOS) — runs when RUN_VOICEOVER is true**

Before starting, tell the user what's needed:

Tell the user: "For the VoiceOver check, please enable VoiceOver (Cmd+F5), open the Caption Panel (VoiceOver Utility → Visuals → Show Caption Panel), and use Safari for best results."

Use `AskUserQuestion` with 1 question:

**Question: "Is VoiceOver ready?"**
- Header: `VoiceOver`
- Option 1 — label: `Ready`, description: `VoiceOver is on and the Caption Panel is visible`
- Option 2 — label: `Skip`, description: `Continue without the VoiceOver check`

If Skip, note it and move on.

Load `references/voiceover-checklist.md`. After each interaction, ask the user to share what the Caption Panel shows. Work through the checklist items in that file.

Tell the user the result:
> VoiceOver check complete — *[N issues / all clear]*
>
> Thanks for your help sharing the Caption Panel output.

#### Step 8: Structure check *(runs when RUN_STRUCTURE is true)*

**Skip if `RUN_STRUCTURE` is false.**

Tell the user:
> Checking heading hierarchy, table markup, and landmark structure...

Load `references/structure-checks.md`. Check headings, tables, and landmark regions via `browser_snapshot` and `browser_evaluate`. Report using the output template in that file.

Tell the user the result:
> Structure check complete — *[N issues / all clear]*

#### Step 9: Form check *(runs when RUN_FORMS is true)*

**Skip if `RUN_FORMS` is false.**

Use `AskUserQuestion` with 1 question:

**Question: "Ready to check form accessibility?"**
- Header: `Forms`
- Option 1 — label: `Proceed`, description: `Submit the form with empty fields to trigger validation and check error handling`
- Option 2 — label: `Skip form submit`, description: `Check form markup only — don't submit the form`
- *(User types a concern via "Other" if needed)*

If Proceed: submit the form empty, then inspect `aria-invalid`, `aria-describedby`, `aria-live` via `browser_evaluate`.
If Skip form submit: check form markup and labels without submitting.

Load `references/form-checks.md`. Submit the form empty, inspect `aria-invalid`, `aria-describedby`, `aria-live` via `browser_evaluate`. Report using the output template.

Tell the user the result:
> Form check complete — *[N issues / all clear]*

#### Step 10: State summary and loop

Summarise findings for this state:

> **Results for: *[state description]***
>
> | Test | Issues |
> |---|---|
> | Axe | N |
> | Keyboard | N |
> | NVDA | N |
> | VoiceOver | N |
> | Structure | N |
> | Forms | N |
>
> *[Only include a row for each test that actually ran this state — omit rows for tests that were not active.]*
>
> *[If issues: brief list of the most impactful findings]*
>
> ---

Use `AskUserQuestion` with 1 question:

**Question: "Would you like to test another page or state?"**
- Header: `Next`
- Option 1 — label: `Add another state`, description: `Navigate in the browser to the next page, dialog, or error state, then confirm`
- Option 2 — label: `Done`, description: `Finish the audit and generate the report`

If Add another state: go back to Step 3. If Done: proceed to Phase 3.

**After recording findings for this state**, save a checkpoint:

Write `<SCREENSHOT_PATH>/a11y-<label-slug>-checkpoint.json` with the current `STATES_TESTED` and `ALL_FINDINGS`. If the file exists, overwrite it. This allows the session to be resumed if Claude's context resets.

```json
{
  "label": "[LABEL]",
  "date": "[today]",
  "screenshot_path": "[SCREENSHOT_PATH]",
  "tests": { "code": true/false, "keyboard": true/false, "nvda": true/false, "voiceover": true/false, "structure": true/false, "forms": true/false, "static": true/false },
  "states_tested": [
    { "label": "[LABEL]", "url": "[FULL_URL]", "findings_count": N }
  ],
  "all_findings": [
    { "url": "[FULL_URL]", ... }
  ]
}
```

Each state record includes `url` (the full URL from `browser_evaluate`) so the CSV can report the correct URL per finding across multi-state sessions.

*(Loop control is handled by the AskUserQuestion above.)*

---

### Phase 3: Fix and report

#### Step 11: Full session summary

Before fixing or reporting, summarise everything tested:

> **Audit complete — here's the full picture**
>
> States tested: *[list each state label]*
>
> Total findings: *N*
> - Critical: N
> - Serious: N
> - Moderate: N
> - Minor: N
>
> *[If findings: top 3–5 most impactful findings across all states]*
> *[If no findings: "No violations were found across any of the tested states. The audit will be recorded as all clear."]*
>
> *[If FIX_VIOLATIONS and findings: "I'll now work through the fixes."]*
> *[If not FIX_VIOLATIONS or no findings: "No code changes will be made — findings are recorded in the audit log."]*

#### Step 12: Fix violations *(runs when FIX_VIOLATIONS is true)*

**Skip if `FIX_VIOLATIONS` is false.** Findings are recorded — no code is changed.

Tell the user:
> Starting fixes — I'll work through each affected file. I'll tell you what I'm changing and why before I save.

Group all findings by file. For each file:

1. Read the full current file contents
2. Store the original content in memory
3. Tell the user: *"Fixing [file] — addressing: [list of violations in this file]"*
4. Plan all fixes for this file at once — batch into a single edit
5. Apply in a single `Edit` or `Write` call
6. If `VISUAL_REVIEW`: run the visual review protocol below

Match fix patterns using `references/fix-patterns.md`. For uncovered cases, derive from the axe `failureSummary` or the [axe rules documentation](https://dequeuniversity.com/rules/axe/).

**Do not suppress violations with `aria-hidden` hacks** — fix the underlying structure.

If a violation cannot be fixed after 3 attempts, tell the user:
> I wasn't able to fix *[violation]* in *[file]* after 3 attempts. I've documented what I tried. You may want to review this one manually.

---

#### Visual review protocol *(when VISUAL_REVIEW is true)*

After saving each file, wait for hot-reload, then:

**1. Screenshot the affected state:**
```
<output-path>/a11y-<label-slug>-<state-slug>-after-pending.png
```

**2. Generate a comparison page:**
```
<output-path>/a11y-compare-<file-slug>-<timestamp>.html
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Visual comparison</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body { font-family: system-ui, sans-serif; background: #111; color: #fff; }
    header { padding: 12px 16px; background: #222; font-size: 13px; }
    header strong { color: #7dd3fc; }
    .grid { display: grid; grid-template-columns: 1fr 1fr; height: calc(100vh - 40px); }
    .pane { display: flex; flex-direction: column; border-right: 1px solid #333; }
    .pane:last-child { border: none; }
    .pane h2 { padding: 8px 12px; font-size: 11px; text-transform: uppercase;
                letter-spacing: .08em; color: #94a3b8; background: #1a1a1a; }
    .pane img { flex: 1; object-fit: contain; width: 100%; }
  </style>
</head>
<body>
  <header>Visual comparison — <strong>[file] · [fixes applied]</strong></header>
  <div class="grid">
    <div class="pane"><h2>Before</h2><img src="[before-path]" alt="Before"></div>
    <div class="pane"><h2>After</h2><img src="[after-pending-path]" alt="After"></div>
  </div>
</body>
</html>
```

**3. Open in Playwright:**
```
browser_navigate → file:///[path to comparison HTML]
```

**4. Ask the user:**

> I've fixed *[N violations]* in *[file]*. The before/after comparison is now open in the browser.
>
> Does this visual change look acceptable?
> - **yes** — keep the fix and move on
> - **no** — I'll revert this file and mark those findings as deferred

**If yes:** rename `after-pending.png` → `after.png`, record as fixed, navigate back, continue.

**If no:** restore file from stored original, delete pending screenshot and comparison HTML, mark findings as "deferred — visual regression", navigate back, continue.

---

#### Step 13: Re-verify *(runs when FIX_VIOLATIONS is true)*

Tell the user:
> Re-running axe to confirm all fixes hold...

Re-run the Step 5 axe script at every checkpoint.

- All clear → proceed
- Violations remain → fix, re-run, repeat
- Persists after 3 attempts → document, tell the user, move on

Tell the user the result:
> Re-verification complete — *[all clear / N remaining issues noted in the log]*

#### Step 14: After screenshots *(runs when FIX_VIOLATIONS is true)*

If `VISUAL_REVIEW` is true, after-screenshots were already captured per file. Confirm all states have an approved `after.png`.

If `VISUAL_REVIEW` is false, capture after-state for every before screenshot:
```
<output-path>/a11y-<label-slug>-<state-slug>-<n>-after.png
```

Tell the user:
> After screenshots saved. Before/after pairs:
> *[list each pair with file paths]*

#### Step 15: Update the audit log

Load `references/audit-log-format.md`. Always write an entry — even if no violations were found.

**If findings exist:** Append all findings from this session. Group by violation type across all states tested — a violation appearing in multiple states is one finding with multiple locations noted. Read existing entries to determine the next finding number. For deferred findings (visual regression or unfixable), use the deferred entry format.

**If no findings:** Add a row to the Audits table noting the date, states tested, tests run, and "No violations found." This creates a record that the audit took place. No numbered findings are added to the Detailed Findings section.

**URL in the Audits table:** Include the full URL (or URLs if multiple states were tested) in the URL column. If multiple states had different URLs, list them separated by semicolons.

Also delete the checkpoint file (`<SCREENSHOT_PATH>/a11y-<label-slug>-checkpoint.json`) now that findings are safely recorded in the audit log.

Tell the user:
> Audit log updated — *[N new findings added (findings #X–#Y) / "no violations found — audit recorded"]*

#### Step 16: Lint *(runs when FIX_VIOLATIONS is true)*

Tell the user:
> Running lint to check for any issues introduced by the fixes...

Run the lint command from Step 1. Flag only errors **new** since the fixes. Do not suppress with disable comments — fix the root cause or report to the user.

Tell the user the result:
> Lint *[passed / found N new issues — [brief description]]*

#### Step 17: CSV export *(runs when EXPORT_REPORT is true)*

Tell the user:
> Generating CSV report...

Check whether a CSV for this label already exists in the screenshots folder:
```
<output-path>/a11y-<label-slug>.csv
```

- **`--fresh-report`:** delete the existing CSV first, then create it fresh with headers
- **`--report` and file exists:** append new findings (do not duplicate existing rows)
- **`--report` and file does not exist:** create it with headers

Load `references/audit-log-format.md` for column definitions. Write one row per finding from `ALL_FINDINGS`. Include static analysis findings as `test_type: static`. Multiple files separated by semicolons.

Tell the user:
> CSV report *[created / updated]*: *[full file path]*
>
> *[N total findings across N states]*

---

#### Step 17.5: Remediation plan *(runs when EXPORT_PLAN is true)*

**Skip if `EXPORT_PLAN` is false.**

Tell the user:
> Generating remediation plan...

Write a Markdown file to:
```
<output-path>/a11y-<label-slug>-remediation-plan.md
```

Use this structure:

```markdown
# Accessibility Remediation Plan — [LABEL]

**Audit date:** [today]
**WCAG target:** [WCAG_TARGET]
**States audited:** [list each label and URL]
**Tests run:** [list active tests]

---

## Summary

| Severity | Count |
|---|---|
| Critical | N |
| Serious | N |
| Moderate | N |
| Minor | N |
| **Total** | **N** |

*[If no findings: "No violations were found across all tested states."]*

---

## Findings

*[Group findings by severity — critical first. For each finding:]*

### [Impact] — [Violation name / rule ID]

**WCAG criterion:** [e.g. 1.3.1 Info and Relationships (Level A)]
**Affected pages:**
- [URL or state label] — [element or selector, e.g. `<input type="checkbox"> inside <button>`]

**What's wrong:** [plain-language description of the problem]

**How to fix:** [concrete fix guidance — what the correct markup or pattern looks like]

**Status:** [Open / Fixed / Deferred — visual regression / Deferred — unfixable after 3 attempts]

---
*[repeat for each finding]*

## Next Steps

*[If open findings remain:]*
- Address critical and serious findings before the next release
- Run `/a11y-audit "[LABEL]" --fix` to apply fixes automatically, or work through the findings manually using the guidance above
- Re-run the audit after fixes to verify

*[If all fixed:]*
- All findings have been addressed in this session
- Run your full test suite and consider committing the changes
- Re-audit after the next significant UI change

*[If no findings:]*
- No violations were found — no action required
- Re-audit after the next significant UI change

---

*Generated by [a11y-audit](https://github.com/lanedigital/a11y-audit) · [today]*
```

Tell the user:
> Remediation plan saved: *[full file path]*
>
> Share this file with your team, attach it to a ticket, or use it to track remediation progress.

---

#### Step 18: Session close

Tell the user:

> **All done — here's a summary of this session**
>
> States audited: *[list]*
> Total findings: *N* (*N fixed, N deferred, N open*)
> *[If no findings: "No violations were found — great result."]*
>
> Files changed: *[list, or "none"]*
> *[If EXPORT_REPORT:]* CSV report: *[path]*
> *[If EXPORT_PLAN:]* Remediation plan: *[path]*
>
> **Next steps:**
> *[If open findings remain:]* "There are *N* open findings in the audit log. Run `/a11y-audit [label] --fix` to address them."
> *[If all fixed:]* "All findings have been addressed. Consider committing the changes and running your full test suite."
> *[If no findings:]* "No violations found. The audit is recorded in *[audit log path]*. Consider running again after the next significant UI change."
> *[If report-only with findings:]* "Findings are recorded in *[audit log path]*. Run with `--fix` when you're ready to address them."
>
> To audit another feature, run `/a11y-audit [new label] [flags]`.

---

## Hard Rules

- **Never navigate the browser without the user's instruction** — the user drives navigation; Claude drives testing
- **Never submit forms or trigger destructive actions** without warning the user first
- **Never save screenshots or temp files inside the project folder**
- **Never save the same source file more than once per fix** — batch all changes into one edit per file
- **Never commit without explicit user instruction**
- **Re-inject axe after every `browser_navigate`** — it does not persist between navigations
- **Never use `--fix` on a project you cannot modify** — if source files are not in the working directory, do not attempt fixes

## Error Handling

**Playwright not connected:** If browser tools are unavailable, stop and tell the user to confirm Playwright MCP is connected (see README for setup).

**Axe CDN unavailable:** Retry once. If it fails again, tell the user — the axe scan cannot proceed. Other tests can still run.

**NVDA not running:** If the user says NVDA isn't ready, offer to skip the NVDA step and continue with other tests.

**VoiceOver not enabled:** If the user says VoiceOver isn't running or the Caption Panel isn't visible, offer to skip the VoiceOver step and continue with other tests.

**Fix fails after 3 attempts:** Tell the user specifically what was tried and why it failed. Mark the finding as deferred and continue.

**Visual review comparison page fails to load:** Report the before/after image paths in the conversation and ask the user to open them manually. Continue the approval flow.

**Static analysis plugin not available and user declines install:** Note that `--static` was skipped, continue with browser tests.

## Success Criteria

- [ ] User guided through each state — knows what's happening and what to do at every step
- [ ] All selected tests run on each state
- [ ] Full session summary presented after all states tested
- [ ] Zero static violations (or all documented) *(if RUN_STATIC)*
- [ ] Axe returns `count: 0` at every checkpoint *(if RUN_CODE and FIX_VIOLATIONS)*
- [ ] All keyboard checks pass *(if RUN_KEYBOARD)*
- [ ] NVDA checklist completed *(if RUN_NVDA)*
- [ ] VoiceOver checklist completed *(if RUN_VOICEOVER)*
- [ ] Structure report produced *(if RUN_STRUCTURE)*
- [ ] Form report produced *(if RUN_FORMS)*
- [ ] All fixes applied and re-verified *(if FIX_VIOLATIONS)*
- [ ] All fixes approved via visual review *(if VISUAL_REVIEW)*
- [ ] Before/after screenshots saved externally *(if FIX_VIOLATIONS)*
- [ ] Audit log updated with all findings from this session
- [ ] Lint passes with no new errors *(if FIX_VIOLATIONS)*
- [ ] CSV report created or updated *(if EXPORT_REPORT)*
- [ ] Remediation plan generated *(if EXPORT_PLAN)*
- [ ] No files committed unless explicitly asked
