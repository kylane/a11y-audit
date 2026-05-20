# Keyboard Navigation Check Protocol

Use this reference during Step 5 of the audit. Perform checks using Playwright MCP `browser_press_key` and `browser_snapshot` to observe focus state.

---

## Core Tab Order Check

For every interactive element on the page:

1. Start with focus on the browser address bar or first focusable element
2. Press `Tab` repeatedly and record each element that receives focus
3. Verify:
   - Every interactive element is reachable (buttons, inputs, links, checkboxes, selects)
   - Focus indicator is **visible** on each element (not just a browser default outline — check design system focus styles)
   - Tab order follows visual/semantic reading order (top-to-bottom, left-to-right for LTR)
   - No elements are reachable by Tab that should not be (e.g. `sr-only` elements with `tabIndex={0}`)
   - No focus traps outside of modal dialogs

**Failure to report:** Element name / selector, what happened (skipped / wrong order / no indicator / unexpected stop)

---

## Dialog Focus Management

For every modal dialog in scope:

**On open:**
1. Trigger the dialog via keyboard (Tab to trigger, Enter/Space to activate)
2. Verify focus moves into the dialog immediately (first focusable element, or explicitly set element)
3. Verify the page behind is inert (Tab should not escape the dialog)

**While open:**
4. Tab through all controls — verify every control is reachable
5. Shift+Tab backward — verify reverse order works
6. Verify the tab cycle stays within the dialog (focus should wrap from last → first element)

**On close:**
7. Press Escape — dialog should close
8. Verify focus returns to the element that triggered the dialog (not `document.body`)
9. Verify the trigger element is still visible and focused

**Failure to report:** Which step failed, what focus did instead

---

## Key Binding Verification

| Key | Expected behaviour | Notes |
|---|---|---|
| `Tab` | Move focus to next interactive element | In natural DOM order |
| `Shift+Tab` | Move focus to previous interactive element | Reverse order |
| `Enter` | Activate button, follow link, submit form | Also opens some listboxes |
| `Space` | Toggle checkbox, activate button | Does NOT follow links |
| `Escape` | Close dialog, dismiss menu, cancel action | Should not navigate away |
| `ArrowUp` / `ArrowDown` | Navigate radio groups, listbox options, menu items | |
| `ArrowLeft` / `ArrowRight` | Navigate tab panels, sliders, some composite widgets | |
| `Home` / `End` | Jump to first/last option in listbox or menu | |

---

## Multi-Step Form Check

For wizard-style forms with multiple steps:

1. Complete Step N by keyboard only
2. Tab to the Next/Continue button and press Enter
3. Verify:
   - Step N+1 content is shown
   - Focus does **not** land on a hidden `sr-only` element mid-page (check for unexpected tab stops)
   - An `aria-live` announcement confirms the step change (audible without focusing the element)
   - All fields in the new step are reachable by Tab

---

## Autocomplete / Combobox Check

1. Tab to the autocomplete input
2. Type a query
3. Verify the dropdown/listbox opens
4. Press ArrowDown to move into options
5. Press Enter to select
6. Verify the selected value appears and focus is managed correctly
7. Press Escape — verify the popup closes and focus returns to the input

---

## Table Navigation Check

For data tables with interactive content:

1. Tab into the table
2. Verify column header cells are announced (screen reader context — use NVDA checklist for this)
3. If the table has row actions (buttons, links), verify they are reachable and the row context is clear
4. For tables with checkboxes in rows, verify each checkbox has a label that includes the row context (e.g. "Select Project Alpha" not just "Select")

---

## WCAG 2.2 Additional Checks

These criteria are new in WCAG 2.2 AA and are not fully covered by axe — verify them manually during the keyboard pass.

### 2.4.11 Focus Appearance

The focus indicator for every keyboard-focusable element must meet all three conditions:
- **Area:** The focus indicator encloses an area at least as large as the perimeter of the unfocused component × 2 CSS pixels (i.e. a 2px solid outline around the full component is the minimum)
- **Contrast:** The focused pixels have at least 3:1 contrast ratio against their unfocused state (e.g. a blue outline appearing over a white background must be 3:1 against white)
- **Not obscured:** The focus indicator is not entirely hidden by other content (e.g. sticky headers, overlays)

Use `browser_snapshot` after tabbing to each element and check the focus style visually. Flag elements where the focus ring is: too thin (< 2px), too low contrast, or clipped by a sticky element.

### 2.5.7 Dragging Movements

Any feature that uses a drag operation (e.g. drag-to-reorder, drag-to-resize, drag-to-upload) must also offer a single-pointer alternative that does not require dragging.

Check: if there is no drag functionality in scope, skip this. If there is, verify a click/tap-based alternative exists for the same action.

### 2.5.8 Target Size (Minimum)

Every pointer/touch target must be at least **24×24 CSS pixels**, OR have sufficient spacing so the total clickable area including spacing meets 24×24px. Exceptions: inline text links, targets sized by the browser default (e.g. native checkboxes).

Use browser DevTools (or `browser_evaluate` to read `getBoundingClientRect()`) to spot-check small targets such as icon buttons, close buttons, and pagination controls.

```js
// Spot-check a specific element's target size
document.querySelector('[aria-label="Close"]').getBoundingClientRect()
// width and height should both be >= 24
```

### 3.3.7 Redundant Entry

Information the user has already entered within the same session should not need to be re-entered, unless re-entry is essential or the info is a security credential.

Check multi-step forms: if a field was filled in an earlier step, it should be pre-populated or auto-completed in later steps where the same data is requested.

### 3.3.8 Accessible Authentication (Minimum)

Authentication steps (login, CAPTCHA, security questions) must not rely solely on a cognitive function test — i.e. there must be an alternative to memory, puzzle-solving, or transcription. Object recognition (e.g. "click all the traffic lights") is acceptable; transcribing distorted characters is not.

Check: if the feature includes login or any auth challenge, verify at least one non-cognitive alternative is available (e.g. copy-paste of a password, a link-based login, or a password manager-compatible input).

---

## Reporting Format

For each keyboard check, report:

```
✅ Tab order — all 12 interactive elements reachable, order matches visual layout
✅ Focus indicator — visible on all elements (blue outline, 2px offset)
✅ Dialog: Add Grants — opens focused on first checkbox, Escape closes, focus returns to "Add grants" button
❌ Dialog: Add Service — Escape key has no effect (missing onClose prop on Dialog)
❌ Step navigation — hidden sr-only div receives Tab focus between steps 1→2 (tabIndex={0} should be {-1})
```
