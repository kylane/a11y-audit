---
name: a11y-audit
description: Run a full WCAG 2.2 AA accessibility audit (axe-core + keyboard + NVDA) on any web app, fix violations, and record findings. Works with React, Vue, Angular, or plain HTML — auto-detects framework, dev server, and lint command.
triggers:
  - "run accessibility audit"
  - "a11y audit"
  - "wcag audit"
  - "accessibility check"
  - "axe audit"
  - "check accessibility"
  - "audit for accessibility"
---

# Accessibility Audit Skill

Run a structured WCAG 2.2 AA accessibility audit against a live dev server using Playwright MCP + axe-core CDN injection. Covers automated scanning at every distinct UI state, keyboard navigation checks, an NVDA manual-check checklist, visual regression guidance, and audit log recording.

Works with any web framework. Auto-detects project structure, component library, dev server URL, and lint command. If a project-specific config exists at `a11y-config.md` in the project root, load it first for overrides.

## What This Skill Does

1. **Detect project context** — framework, component library, dev server, lint command, audit log location
2. **Baseline screenshots** — capture before-state of all in-scope pages and dialogs (saved outside the repo)
3. **Axe scan** — inject axe-core 4.10.3 at every distinct UI state; report violations by impact
4. **Keyboard check** — tab order, Escape-to-close, focus return, dialog behaviour
5. **NVDA checklist** — generate a manual screen-reader test script tailored to the feature
6. **Fix violations** — apply targeted fixes using framework-appropriate patterns (load `references/fix-patterns.md`)
7. **Re-verify** — re-run axe at every checkpoint; confirm zero violations
8. **After screenshots** — capture post-fix state for visual regression comparison
9. **Audit log** — record findings in the project's audit log (or create one if none exists)
10. **Lint** — run the detected lint command; confirm no new errors

## Skill Invocation

This skill is triggered when the user invokes `/a11y-audit [feature-or-page]`.

The argument (optional) names the feature or page to audit. If no argument is given, audit whatever page is currently open in the browser.

---

## Implementation

### Step 0: Load project config if it exists

Check for `a11y-config.md` in the project root. If found, read it — it contains project-specific overrides for dev server URL, auth instructions, audit log path, screenshot output path, and component library. If not found, proceed with auto-detection below.

Also load `references/fix-patterns.md` now — you will need it during the fix phase.

### Step 1: Auto-detect project context

Read `package.json` to determine:

**Framework:**
- `react` / `react-dom` → React
- `vue` → Vue
- `@angular/core` → Angular
- `svelte` → Svelte
- None of the above → Generic HTML

**Component library** (check `dependencies` and `devDependencies`):
- `@mui/material` → MUI (Material UI)
- `@chakra-ui/react` → Chakra UI
- `@radix-ui/*` → Radix UI
- `@headlessui/react` or `@headlessui/vue` → Headless UI
- `primereact` / `primevue` → PrimeReact / PrimeVue
- `antd` → Ant Design
- None matched → Generic / custom components

**Dev server URL** (in order of priority):
1. Check `a11y-config.md` (if it existed)
2. Check `vite.config.ts` / `vite.config.js` for `server.port`
3. Check `next.config.js` for custom port
4. Check `package.json` scripts for `--port` flag in `dev` / `start` script
5. Default: `http://localhost:3000`

**Lint command** (check `package.json` scripts for, in order):
- `lint` → use `npm run lint` / `yarn lint` / `pnpm lint` (match package manager)
- `lint:web`, `lint:app`, `lint:frontend` → use that specific script
- No lint script → skip lint step and note it

**Package manager:**
- `yarn.lock` present → yarn
- `pnpm-lock.yaml` present → pnpm
- `package-lock.json` present → npm

**Existing audit log** (check in order):
- `docs/ux/AUDIT_LOG.md`
- `docs/AUDIT_LOG.md`
- `AUDIT_LOG.md`
- `.accessibility/audit-log.md`
- None found → create `docs/accessibility/AUDIT_LOG.md` using the template in `references/audit-log-format.md`

**Screenshot output path:**
- If `a11y-config.md` specifies a path, use it
- Windows: `C:\Users\<username>\Pictures\a11y-screenshots\` (derive username from environment)
- macOS/Linux: `~/Pictures/a11y-screenshots/` or `~/Desktop/a11y-screenshots/`
- **NEVER save screenshots inside the project folder** — they pollute git and may trigger hot-reload file watchers

Report the detected context to the user before proceeding:
```
Detected: React + MUI · Dev server: http://localhost:3000 · Lint: yarn lint
Audit log: docs/ux/AUDIT_LOG.md · Screenshots → C:\Users\<user>\Pictures\a11y-screenshots\
```

### Step 2: Navigate to the target page

**NEVER guess or type route paths directly.** Direct URL navigation bypasses auth and produces blank pages or 401 errors in most apps.

Preferred approach:
1. Use `browser_navigate` to go to the dev server home page (e.g. `http://localhost:3000`)
2. If the page is blank or errors, stop and ask the user to confirm the dev server is running
3. Use `browser_click` to navigate to the target page via the app's **own navigation** (sidebar, nav menu, breadcrumbs, links)
4. Use `browser_click` to open dialogs, navigate wizard steps, and trigger the states you need to audit

If a route path must be verified: check the project's route config file (e.g. `src/routes.tsx`, `src/router/index.ts`, `app/routes.ts`, `pages/` directory for Next.js) — then still navigate via the app UI, not by typing the URL.

**Auth:** If the app requires login:
- Check `a11y-config.md` for auth instructions specific to this project
- Otherwise ask the user which account to use, or ask them to log in and confirm when ready
- Prefer a super-user/admin account that can access all areas being tested

### Step 3: Baseline screenshots

Before touching any code, capture the before-state of every page and dialog in scope.

Use `browser_take_screenshot` for each state and save to the external screenshots folder determined in Step 1:
```
<output-path>/a11y-<feature>-<state>-before.png
```

Example names: `a11y-create-record-step1-before.png`, `a11y-grants-dialog-open-before.png`

States to capture: page at rest, each dialog open, each wizard step, error state (trigger validation errors if applicable).

### Step 4: Inject axe and scan at every distinct UI state

Use `browser_evaluate` with this script. **Re-inject after every `browser_navigate`** — axe does not persist between navigations.

```js
const script = document.createElement('script');
script.src = 'https://cdnjs.cloudflare.com/ajax/libs/axe-core/4.10.3/axe.min.js';
document.head.appendChild(script);
await new Promise(resolve => script.onload = resolve);
const results = await axe.run(document, {
  runOnly: { type: 'tag', values: ['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa', 'wcag22aa'] }
});
return {
  violations: results.violations.map(v => ({
    id: v.id,
    impact: v.impact,
    description: v.description,
    nodes: v.nodes.length,
    failureSummary: v.nodes[0]?.failureSummary,
    html: v.nodes[0]?.html
  })),
  count: results.violations.length
};
```

**Scan at each distinct UI state** — not once per page:
- Page at rest (no dialogs or overlays)
- Each dialog / modal open individually
- Each step of a multi-step wizard or form
- Error state (submit with empty required fields)
- After closing a dialog (verify DOM is clean)

**Report violations grouped by impact:** `critical` → `serious` → `moderate` → `minor`

For each violation report:
- Axe rule ID
- WCAG success criterion
- Impact level
- Number of affected nodes
- First failing HTML snippet
- Preliminary fix guidance (reference `references/fix-patterns.md`)

If all checkpoints return `count: 0` — note "Zero violations" and proceed to keyboard checks.

### Step 5: Keyboard navigation check

Load `references/keyboard-checks.md` for the full protocol. At minimum, verify:

- Every interactive element is reachable by Tab
- Focus indicator is visible on each focused element
- Tab order matches visual/semantic reading order
- No focus traps (Tab or Escape should always allow escape from any context)
- Dialogs: focus enters on open, Escape closes, focus returns to trigger on close
- Multi-step forms: advancing steps does not create unexpected tab stops

Use `browser_press_key` with key values: `Tab`, `Shift+Tab`, `Enter`, `Space`, `Escape`, `ArrowUp`, `ArrowDown`.

Report any keyboard failures with: element description, key pressed, expected vs actual behaviour.

### Step 6: NVDA screen reader check

Load `references/nvda-checklist.md` for the full protocol, expected announcement formats, and the checklist template.

Two automation approaches are available — use whichever is set up in the project. `nvda-testing-driver` is preferred when available; Speech Viewer is the fallback.

---

**Approach A — `nvda-testing-driver` (preferred)**

`nvda-testing-driver` is an npm package that connects to NVDA via its Controller Client interface and exposes a programmatic API for starting NVDA, sending key commands, and capturing spoken output as strings — no Speech Viewer window required.

Check whether it is installed:
```bash
cat package.json | grep nvda-testing-driver
# or check devDependencies in package.json
```

If installed, use it to:
1. Start NVDA via the driver API
2. Navigate the feature using the driver's key command methods (Tab, arrow keys, Enter, Escape)
3. Capture the spoken text returned after each interaction
4. Compare captured text against the expected announcement formats in `references/nvda-checklist.md`
5. Record pass/fail per checklist item with the actual captured string

Refer to the package documentation for exact API method names — they can vary across versions. The driver returns spoken text as strings or arrays that can be directly compared or logged.

If `nvda-testing-driver` is not installed and the project uses npm/yarn/pnpm, ask the user whether to install it as a dev dependency before proceeding, or fall back to Approach B.

---

**Approach B — NVDA Speech Viewer (fallback)**

Ask the user to confirm before proceeding:
1. NVDA is running (system tray icon visible)
2. Speech Viewer is open: NVDA menu → Tools → Speech Viewer
3. The Speech Viewer window is visible alongside the browser

Use `browser_press_key` and `browser_click` to navigate the feature. After each interaction, ask the user to share the current Speech Viewer output. Assess each announcement against the expected formats in `references/nvda-checklist.md` and record pass/fail.

---

Adapt the checklist from `references/nvda-checklist.md` to the specific feature and print the completed results to the conversation.

### Step 7: Fix violations

**Before editing any file:**
1. Read `references/fix-patterns.md` — it contains patterns organised by WCAG rule and by component library
2. Read the full current contents of each file before editing
3. Plan all changes for a file, then apply in a single `Edit` or `Write` call — **never save the same file more than once per task** (multiple saves trigger hot-reload and can crash the app or destroy browser state)

Match fix patterns to the detected component library. If the library is not covered in `references/fix-patterns.md`, derive the fix from the axe `failureSummary` and the [axe rules documentation](https://dequeuniversity.com/rules/axe/).

**Do not suppress violations with `aria-hidden` hacks** — fix the underlying structure.

### Step 8: Re-verify — axe at every checkpoint

After all fixes are applied, re-run the axe scan (Step 4 script) at every checkpoint identified earlier.

- All `count: 0` → proceed to after screenshots
- Violations remain → fix, re-run, repeat
- Same violation persists after 3 attempts → stop fixing it, document what was tried, report to the user

### Step 9: After screenshots

Capture after-state matching every before screenshot:
```
<output-path>/a11y-<feature>-<state>-after.png
```

Report the full list of before/after pairs to the user. Call out any visual differences explicitly — even small spacing changes may affect the design review.

### Step 10: Update the audit log

Load `references/audit-log-format.md` for the format. Append findings to the detected audit log (or the newly created one from Step 1).

Group findings by violation type, not by file — systemic violations affecting multiple files are one finding. Read existing entries first to determine the correct next finding number.

### Step 11: Lint

Run the lint command detected in Step 1. Compare output to the pre-audit baseline — only flag errors that are **new** since the fixes were applied. Do not suppress new lint errors with disable comments; fix the root cause or report to the user.

---

## Hard Rules (apply in all projects)

- **NEVER guess route paths** — always navigate via the app's own UI
- **NEVER save screenshots or temp files inside the project folder** — use external paths only
- **NEVER save the same file more than once per task** — batch all edits into one save per file
- **NEVER commit without explicit user instruction**
- **Re-inject axe after every `browser_navigate`** — it does not persist between navigations

## Error Handling

**Dev server not running:** If the home page returns a connection error, stop and ask the user to start the dev server. Do not proceed.

**Auth required:** If navigation hits a login wall, check `a11y-config.md` for auth instructions, or ask the user which account to use and wait for them to confirm they're logged in.

**Axe CDN unavailable:** Retry once. If it fails again, report to the user — the audit cannot proceed without axe.

**Violation persists after 3 fix attempts:** Document what was tried and report to the user with the axe rule, the HTML snippet, and the approaches attempted.

**Lint fails after fix:** Do not suppress. Fix the root cause or report to the user.

## Success Criteria

- [ ] Axe returns `count: 0` at every checkpoint
- [ ] All keyboard checks pass
- [ ] NVDA checklist printed for the tester
- [ ] Before/after screenshots saved externally and reported
- [ ] Audit log updated with numbered findings
- [ ] Lint passes with no new errors
- [ ] No files committed (unless user explicitly asked)
