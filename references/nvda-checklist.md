# NVDA Screen Reader Check

Use this reference during Step 6 of the audit.

**Baseline environment:** NVDA + Firefox. Secondary: NVDA + Chrome.

---

## Approach A — `nvda-testing-driver` (preferred)

`nvda-testing-driver` is an npm package that connects to NVDA via its Controller Client interface. It provides a programmatic API for starting NVDA, sending key commands, and capturing spoken output as strings — fully automated, no Speech Viewer window required.

**Installation (dev dependency):**
```bash
npm install --save-dev nvda-testing-driver
# or: yarn add -D nvda-testing-driver
# or: pnpm add -D nvda-testing-driver
```

**General workflow:**

```js
// Pseudocode — refer to package docs for exact method names
import { nvdaTestingDriver } from 'nvda-testing-driver'

// 1. Start NVDA
await nvdaTestingDriver.start()

// 2. Navigate to the page (Playwright controls the browser in parallel)
// browser_navigate / browser_click happen here

// 3. Send a key command and capture what NVDA says
await nvdaTestingDriver.sendKeys(['Tab'])
const spoken = await nvdaTestingDriver.getSpokenText()
// spoken might be: "Lead Investigator, edit, required"

// 4. Assert against expected format
// e.g. expect(spoken).toContain('Lead Investigator')

// 5. Stop NVDA when done
await nvdaTestingDriver.stop()
```

**Integration pattern with Playwright MCP:**
- Playwright MCP controls the browser (navigation, clicks, key presses)
- `nvda-testing-driver` runs in a Node script alongside, capturing what NVDA announces in response to Playwright's actions
- After each Playwright interaction, call the driver's spoken-text method and assess the output
- Log the captured string next to each checklist item

**Key things to capture and assess:**
- Focus events (Tab → what does NVDA announce for the newly focused element?)
- State changes (checking a checkbox → does NVDA announce "checked"?)
- Dialog open/close (does the dialog title get announced?)
- Live region updates (do step changes and errors announce automatically?)

Refer to the package's README and changelog for the exact API — method names and return shapes can vary across versions.

---

## Approach B — Speech Viewer (fallback)

Use when `nvda-testing-driver` is not available or not installed.

NVDA's Speech Viewer (NVDA menu → Tools → Speech Viewer) shows everything NVDA reads as visible text in a floating window.

**Setup:**
1. Start NVDA (system tray icon should appear)
2. Open Speech Viewer: NVDA menu → Tools → Speech Viewer
3. Position the Speech Viewer window alongside the browser
4. Confirm with the user that both are visible before proceeding

**Automated workflow:**
- Use `browser_press_key` and `browser_click` to navigate (Tab, arrow keys, Enter, Escape)
- After each interaction, ask the user to share the current Speech Viewer output
- Assess each announcement against the expected format below
- Record pass/fail per checklist item

**Expected announcement formats:**

| Element | Expected NVDA output |
|---|---|
| Page load | `"[page title]"` |
| Text field | `"[label], edit, [required if applicable]"` |
| Checkbox | `"[label], checkbox, checked"` / `"unchecked"` |
| Button | `"[label], button"` |
| Link | `"[label], link"` |
| Select / combobox | `"[label], [selected value], combo box"` |
| Dialog open | `"[dialog title], dialog"` |
| Live region update | Announcement spoken automatically without focus |
| Table cell | `"[column header], [cell value]"` |
| Error message | Announced via live region or focus shift |

Deviations from these formats (e.g. `"button"` alone, a 100+ char sentence as a field name, `"dialog"` with no title) are findings.

---

## Checklist Template

Adapt this for the specific feature being audited:

1. Replace `[feature]` with the page or flow name
2. Add items for each dialog, table, or dynamic region present
3. Remove sections that do not apply
4. Record the actual Speech Viewer output next to each item

---

```
NVDA Speech Viewer Check — [Feature Name]
Date: [YYYY-MM-DD] | Environment: NVDA [version] + Firefox [version]
Branch / Build: _____________
Method: Speech Viewer (automated via Playwright)

BROWSE MODE
-----------------------------------
[ ] H key — heading hierarchy logical (h1 → h2 → h3, no skips)
    Actual: ___________________________________________
[ ] D key — landmarks present: banner, navigation, main
    Actual: ___________________________________________
[ ] Page title announced on load
    Actual: ___________________________________________

FORM FIELDS
-----------------------------------
[ ] Each field: "[label], [type], [required if applicable]"
    — NOT placeholder text as label
    — NOT a 100+ character sentence as label
    Field 1 ([name]): _________________________________
    Field 2 ([name]): _________________________________
[ ] hint text / field description announced (aria-describedby)
    Actual: ___________________________________________
[ ] Validation error announced without requiring focus
    Actual: ___________________________________________

INTERACTIVE CONTROLS
-----------------------------------
[ ] Buttons: "[label], button" (not "button" alone)
    Actual: ___________________________________________
[ ] Icon-only buttons: meaningful label announced
    Actual: ___________________________________________
[ ] Checkboxes: "[label], checkbox, checked/unchecked"
    Actual: ___________________________________________

DIALOGS (repeat for each dialog in scope)
-----------------------------------
[ ] [Dialog name] — opens as: "[title], dialog"
    Actual: ___________________________________________
[ ] [Dialog name] — focus enters dialog on open
[ ] [Dialog name] — Escape closes, focus returns to trigger
    Actual after close: ________________________________

DYNAMIC CONTENT
-----------------------------------
[ ] Loading state announced
    Actual: ___________________________________________
[ ] Success/error messages announced automatically (aria-live)
    Actual: ___________________________________________
[ ] Step/page changes announced automatically
    Actual: ___________________________________________

TABLES (if applicable)
-----------------------------------
[ ] Column header context included per cell: "[header], [value]"
    Actual: ___________________________________________
[ ] Row action buttons include row context
    Actual: ___________________________________________

FINDINGS
-----------------------------------
Issue 1: _____________________________________________
Issue 2: _____________________________________________
Issue 3: _____________________________________________

Verdict:  [ ] PASS — all announcements appropriate
          [ ] PASS WITH NOTES — minor issues, accepted
          [ ] FAIL — inappropriate or missing announcements found
```

---

## Checklist Template

```
NVDA Screen Reader Checklist — [Feature Name]
Date: [YYYY-MM-DD] | Tester: _____________
Environment: NVDA [version] + Firefox [version]
Branch / Build: _____________

BROWSE MODE (default on page load)
-----------------------------------
[ ] H key cycles through headings — heading hierarchy is logical (h1 → h2 → h3, no skips)
[ ] D key cycles through landmarks — page has: banner, navigation, main, [contentinfo if applicable]
[ ] Page title announced on load — tab title or h1 is meaningful
[ ] No orphaned text — all visible text is part of a readable landmark or element

FORM FIELDS
-----------------------------------
[ ] Each form field announces: label, type (text field / checkbox / select / etc.), required state
[ ] Placeholder text is NOT used as the only label (it disappears on input)
[ ] hint text / field description is associated and announced (aria-describedby)
[ ] Error messages are announced — either via aria-live or by focusing the error

INTERACTIVE CONTROLS
-----------------------------------
[ ] All buttons announce their label (not just "button" or icon name)
[ ] Icon-only buttons have an accessible label (different from the icon's visual description)
[ ] Toggle buttons announce their state (pressed / not pressed)
[ ] Checkboxes announce: label + checked/unchecked state
[ ] Links announce their destination context (not just "click here")

DIALOGS
-----------------------------------
[ ] Opening a dialog announces the dialog title (not "dialog" alone)
[ ] Focus enters the dialog on open
[ ] Tab cycles within the dialog (does not escape to page behind)
[ ] Escape key closes the dialog
[ ] Closing the dialog returns focus to the trigger element and announces return context

[Add one block per dialog in scope:]
[ ] [Dialog name] — title announced: ______________________
[ ] [Dialog name] — all controls reachable and labelled: ______________________

DYNAMIC CONTENT
-----------------------------------
[ ] Loading states announced ("Loading..." or spinner with aria-label)
[ ] Success/error messages announced without requiring focus (aria-live)
[ ] Step/page changes in multi-step forms announced automatically
[ ] Filter/search results update announced (item count or "No results")

TABLES (if applicable)
-----------------------------------
[ ] Column headers announced when navigating cells (Tab through cells)
[ ] Row context is understandable without visual layout (e.g. "Project Alpha, Status: Active")
[ ] Interactive cells (checkbox rows, action buttons) announce their row context

LISTS (if applicable)
-----------------------------------
[ ] List items announced with count ("List with N items")
[ ] Each item announced with its full label (not truncated)
[ ] Selectable list items announce selected state (checkbox / aria-selected)

OVERALL
-----------------------------------
[ ] No content is visually visible but completely missing from the SR reading order
[ ] No content is announced that is not visually present (ghost text)
[ ] Reading order (virtual cursor, arrow keys) matches visual reading order

FINDINGS
-----------------------------------
Issue 1: _____________________________________________
Issue 2: _____________________________________________
Issue 3: _____________________________________________

Verdict:  [ ] PASS — no blocking issues
          [ ] PASS WITH NOTES — minor issues, accepted
          [ ] FAIL — blocking issues found (list above)
```

---

## Common NVDA Findings and What They Mean

| Finding | Likely cause | Fix direction |
|---|---|---|
| Dialog announces "dialog" with no title | `aria-labelledby` missing or pointing to wrong element | Add `aria-labelledby` pointing to dialog heading |
| Button announces "button" only | Button has no text content and no `aria-label` | Add `aria-label` or visible text |
| Field announces placeholder as label | No `<label>` or `aria-label` — only `placeholder` | Add explicit label |
| List item announces full sentence | Label text is too long — description mixed into the label | Shorten label; move descriptive text to a linked hint via `aria-describedby` |
| Dialog doesn't close on Escape | No Escape key handler on the dialog | Add a keydown listener or `onClose` handler wired to Escape |
| Step change not announced | No `aria-live` region for step announcements | Add live region to step announcer element |
| Table column context missing | `<th>` cells not associated via `scope` or `id/headers` | Add `scope="col"` to `<th>` elements |
| Icon button announces icon name | SVG or icon element title is read instead of the intended label | Add `aria-label` on the button and `aria-hidden="true"` on the icon element |
