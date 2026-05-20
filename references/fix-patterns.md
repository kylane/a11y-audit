# Accessibility Fix Patterns

Patterns organised by axe rule ID. Examples use generic HTML and framework-agnostic React — apply the same principle to whichever component library your project uses.

**Target standard: WCAG 2.2 AA**

> **Note — WCAG 2.2 change:** Success criterion 4.1.1 Parsing has been removed from WCAG 2.2. Axe may still report `duplicate-id` and similar parsing rules as informational, but they are no longer a conformance requirement under WCAG 2.2. They remain worth fixing for robustness but should not be counted as AA failures.

---

## `nested-interactive` — Interactive control inside another interactive control

**WCAG:** 4.1.2 Name, Role, Value | **Impact:** Serious

Caused by placing a checkbox, radio, button, or input inside a clickable container (a button, link, or element with `role="button"`). Assistive technologies cannot reliably interact with both.

**HTML:**
```html
<!-- ❌ Wrong — checkbox nested inside a button-role container -->
<div role="button" onclick="toggle()">
  <input type="checkbox" aria-hidden="true" tabindex="-1" />
  <span>Item label</span>
</div>

<!-- ✅ Correct — label wraps a single interactive control -->
<label>
  <input type="checkbox" onchange="toggle()" />
  Item label
</label>
```

**React:**
```tsx
// ❌ Wrong — checkbox inside a clickable wrapper component
<ClickableRow onClick={() => toggle(item)}>
  <input type="checkbox" aria-hidden="true" tabIndex={-1} />
  <span>{item.label}</span>
</ClickableRow>

// ✅ Correct — label as the outer element, single interactive control
<label style={{ display: 'flex', alignItems: 'center', cursor: 'pointer' }}>
  <input type="checkbox" checked={isSelected(item)} onChange={() => toggle(item)} />
  <span>{item.label}</span>
</label>
```

**Principle for component libraries:** Avoid wrapping a checkbox, radio, or input inside any component that renders a button or link. The fix in any library is the same — use a `<label>` as the outer interactive element, with the input inside it.

---

## `list` — Invalid direct children of list element

**WCAG:** 1.3.1 Info and Relationships | **Impact:** Moderate

`<ul>` and `<ol>` must only contain `<li>` elements (or elements with `role="listitem"`). Dividers, wrappers, or non-list elements as direct children fail this rule.

```html
<!-- ❌ Wrong — hr or div as direct child of ul -->
<ul>
  <li>Item 1</li>
  <hr />
  <li>Item 2</li>
</ul>

<!-- ✅ Correct — use CSS border on li instead -->
<ul>
  <li style="border-bottom: 1px solid currentColor;">Item 1</li>
  <li>Item 2</li>
</ul>
```

**Component library note:** If your library renders a `<Divider>` component as a direct child of a list, replace it with a CSS border on the `<li>` element itself. The exact prop varies by library — check your library's list item docs for a `divider` or `border` prop.

---

## `dialog-name` — Dialog has no accessible name

**WCAG:** 4.1.2 Name, Role, Value | **Impact:** Serious

Every `role="dialog"` must have an accessible name via `aria-labelledby` (preferred) or `aria-label`.

**Common cause 1 — Missing `aria-labelledby`:**
```html
<!-- ❌ Wrong -->
<div role="dialog" aria-modal="true">
  <h2>Add item</h2>
</div>

<!-- ✅ Correct -->
<div role="dialog" aria-modal="true" aria-labelledby="dialog-title">
  <h2 id="dialog-title">Add item</h2>
</div>
```

**Common cause 2 — Auxiliary element inside the title contaminates the accessible name:**

ARIA's accessible name computation traverses all children of the element referenced by `aria-labelledby`, including any nested interactive elements. A help button or icon inside the title element can append unexpected text to the dialog name (e.g. "Add item help" instead of "Add item").

```html
<!-- ❌ Wrong — help button inside the title element -->
<h2 id="dialog-title">
  Add item
  <button aria-label="Help">?</button>
</h2>

<!-- ✅ Correct — move auxiliary controls outside the title element -->
<div style="display: flex; align-items: center; gap: 8px;">
  <h2 id="dialog-title" style="flex: 1;">Add item</h2>
  <button aria-label="Help">?</button>
</div>
```

---

## `dialog-name` + missing close handler — Dialog not keyboard-dismissible

**WCAG:** 2.1.2 No Keyboard Trap | **Impact:** Serious

A dialog without an Escape key handler traps keyboard users. They cannot dismiss it unless an explicit close button exists and is reachable.

**React (component library with `onClose` prop):**
```tsx
// ❌ Wrong — Escape key does nothing
<Dialog open={open}>

// ✅ Correct — wire up a close handler
<Dialog open={open} onClose={handleClose}>
```

**Plain HTML / vanilla JS:**
```js
// ✅ Listen for Escape on the dialog element
dialog.addEventListener('keydown', (e) => {
  if (e.key === 'Escape') closeDialog()
})
```

**Backdrop click on data-entry dialogs:**

For dialogs with significant form input, you may want to prevent backdrop click while still allowing Escape. This avoids accidental data loss while keeping keyboard dismissal:

```tsx
// React — distinguish reason when the library provides it
<Dialog
  open={open}
  onClose={(event, reason) => {
    if (reason === 'backdropClick') return
    handleClose()
  }}
>
```

Use judgment: lightweight selection dialogs can allow backdrop click. Heavy form dialogs (multiple fields, file uploads) should disable it.

---

## `label` — Form field has no accessible label, or label is excessively long

**WCAG:** 1.3.1, 2.4.6 | **Impact:** Critical / Moderate

**Missing label:**
```html
<!-- ❌ Wrong -->
<input type="text" placeholder="Search..." />

<!-- ✅ Correct — explicit label element -->
<label for="search">Search</label>
<input id="search" type="text" placeholder="Search..." />

<!-- ✅ Correct — aria-label when a visible label is not possible -->
<input type="text" aria-label="Search" placeholder="Search..." />
```

**Label too long (sentence used as field name):**

Screen readers announce the label on every focus. A 100+ character sentence is disorienting. Use a concise label and move the description to a linked hint element.

```html
<!-- ❌ Wrong — full sentence as label -->
<label for="owner">
  The person responsible for approving the record — must be the named lead on the project
</label>
<input id="owner" type="text" />

<!-- ✅ Correct — concise label, description in a linked hint -->
<label for="owner">Record owner</label>
<input id="owner" type="text" aria-describedby="owner-hint" />
<span id="owner-hint">Must be the named lead on the project</span>
```

---

## `scrollable-region-focusable` — Scrollable region not keyboard-reachable

**WCAG:** 2.1.1 Keyboard | **Impact:** Serious

A scrollable container that is not focusable cannot be scrolled by keyboard-only users.

```html
<!-- ❌ Wrong -->
<div style="overflow-y: scroll; max-height: 300px;">...</div>

<!-- ✅ Correct -->
<div
  style="overflow-y: scroll; max-height: 300px;"
  tabindex="0"
  role="region"
  aria-label="Results list"
>...</div>
```

---

## `color-contrast` — Insufficient colour contrast

**WCAG:** 1.4.3 Contrast (Minimum) | **Impact:** Serious

Normal text requires a 4.5:1 contrast ratio against its background. Large text (18pt / 14pt bold) requires 3:1.

```css
/* ❌ Wrong — light grey text on white, typically ~2.3:1 */
.hint { color: #aaaaaa; }

/* ✅ Correct — darker grey that meets 4.5:1 on white */
.hint { color: #767676; }
```

To check a colour: use the [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/) or the Accessibility panel in browser DevTools. Enter the foreground and background colours to get the ratio.

**Common pitfall — hardcoded colours in component props:** Some component libraries accept a `color` prop that maps to internal theme tokens. If a token's foreground/background pairing doesn't meet contrast, fix it at the theme level rather than overriding per-component — that way the fix applies everywhere.

---

## `image-alt` — Image missing alt text

**WCAG:** 1.1.1 Non-text Content | **Impact:** Critical

```html
<!-- ❌ Wrong -->
<img src="chart.png" />

<!-- ✅ Correct — descriptive alt -->
<img src="chart.png" alt="Bar chart showing quarterly revenue, Q4 highest at $2.3M" />

<!-- ✅ Correct — decorative image (ignored by screen readers) -->
<img src="divider.png" alt="" role="presentation" />
```

In React/JSX: `<img alt="" />` (empty string, not omitted) for decorative images.

---

## `button-name` — Button has no accessible name

**WCAG:** 4.1.2 | **Impact:** Critical

```html
<!-- ❌ Wrong — icon button with no text or label -->
<button onclick="close()">
  <svg>...</svg>
</button>

<!-- ✅ Correct — aria-label -->
<button onclick="close()" aria-label="Close">
  <svg aria-hidden="true">...</svg>
</button>

<!-- ✅ Correct — visually hidden text (announced, not displayed) -->
<button onclick="close()">
  <svg aria-hidden="true">...</svg>
  <span class="sr-only">Close</span>
</button>
```

---

## `aria-hidden-focus` — Focusable element hidden from accessibility tree

**WCAG:** 4.1.2 | **Impact:** Serious

An element with `aria-hidden="true"` that is also focusable creates a confusing experience — keyboard focus lands on it but screen readers cannot read it.

```tsx
// ❌ Wrong — aria-hidden but still focusable
<div aria-hidden="true" tabIndex={0}>Hidden content</div>

// ✅ Option A — remove from tab order too
<div aria-hidden="true" tabIndex={-1}>Hidden content</div>

// ✅ Option B — don't hide it at all, just remove from tab order
<div tabIndex={-1}>Content for programmatic focus only</div>
```

---

## `region` — Content not within a landmark

**WCAG:** 1.3.6 | **Impact:** Moderate

Major content areas should be wrapped in landmark elements to aid screen reader navigation.

```html
<!-- ✅ Use semantic landmarks -->
<header>...</header>
<nav aria-label="Primary navigation">...</nav>
<main>...</main>
<footer>...</footer>

<!-- ✅ Or explicit ARIA roles where semantic elements are not available -->
<div role="main">...</div>
<div role="navigation" aria-label="Primary">...</div>
```

---

## `scope-attr-valid` / empty table header — Column header has no accessible name

**WCAG:** 1.3.1 | **Impact:** Moderate

Table header cells (`<th>`) must have visible text or an `aria-label`. An empty `<th>` leaves the column without context for screen reader users.

```html
<!-- ❌ Wrong — empty header cell -->
<th></th>

<!-- ✅ Correct — explicit label -->
<th aria-label="Actions"></th>
```

---

## `tabindex` — Positive tabindex breaks natural tab order

**WCAG:** 2.4.3 Focus Order | **Impact:** Moderate

Positive `tabindex` values (1, 2, 3...) override the natural DOM order and confuse keyboard users. Only ever use `0` (in tab order, natural position) or `-1` (programmatic focus only, not in tab order).

```html
<!-- ❌ Wrong -->
<button tabindex="3">Submit</button>

<!-- ✅ Correct — rely on DOM order -->
<button>Submit</button>

<!-- ✅ Correct — programmatic focus target not in tab order -->
<div tabindex="-1" id="error-summary">...</div>
```

---

## `aria-live` — Dynamic content not announced

Not an axe rule — caught during keyboard/SR checks.

Dynamic content that updates without a page reload (validation errors, step changes, notifications) must be announced to screen reader users via a live region.

```html
<!-- ✅ Polite — waits for user to finish current interaction -->
<div aria-live="polite" aria-atomic="true" id="status">
  <!-- inject status messages here -->
</div>

<!-- ✅ Assertive — interrupts immediately (use sparingly, for critical errors only) -->
<div aria-live="assertive" id="error-alert">
  <!-- inject critical errors here -->
</div>
```

The live region must be in the DOM **before** content is injected — not created dynamically at announcement time.

Common locations: form validation summary, step/page change announcements, toast/snackbar messages, loading state changes.

---

## WCAG 2.2 AA — New criteria

The following patterns address success criteria introduced in WCAG 2.2. Axe-core covers some partially; others require manual verification during the keyboard check (see `keyboard-checks.md`).

---

## 2.4.11 Focus Appearance — Insufficient focus indicator

**WCAG:** 2.4.11 Focus Appearance | **Impact:** Serious
**Note:** Axe covers this partially. Verify visually during keyboard pass.

The focus indicator must:
- Enclose an area at least equal to a 2 CSS pixel perimeter around the component
- Have at least 3:1 contrast between focused and unfocused appearance
- Not be fully obscured by other content

```css
/* ❌ Wrong — 1px outline, likely fails area requirement */
:focus { outline: 1px solid blue; }

/* ❌ Wrong — outline removed entirely */
:focus { outline: none; }

/* ✅ Correct — 2px solid outline with offset meets area + contrast requirements */
:focus-visible {
  outline: 2px solid #005fcc;
  outline-offset: 2px;
}
```

**Check for sticky header clipping:** If a focused element scrolls under a sticky header, the focus indicator may be fully obscured — a violation. Fix with `scroll-margin-top` matching the header height:

```css
:focus-visible {
  scroll-margin-top: 64px; /* match sticky header height */
}
```

---

## 2.5.7 Dragging Movements — No single-pointer alternative

**WCAG:** 2.5.7 Dragging Movements | **Impact:** AA
**Note:** Only applies if the feature uses drag-and-drop or drag-to-resize.

Any drag operation must have an equivalent single-pointer (click/tap) alternative.

```html
<!-- ❌ Wrong — drag-to-reorder with no alternative -->
<li draggable="true" ondragend="reorder(event)">Item</li>

<!-- ✅ Correct — add keyboard/click-based reorder controls alongside drag -->
<li draggable="true" ondragend="reorder(event)">
  <button aria-label="Move up" onclick="moveUp(index)">↑</button>
  <button aria-label="Move down" onclick="moveDown(index)">↓</button>
  Item content
</li>
```

---

## 2.5.8 Target Size (Minimum) — Touch/pointer target too small

**WCAG:** 2.5.8 Target Size (Minimum) | **Impact:** AA
**Note:** Axe does not currently flag this. Verify manually.

Every interactive target must be at least **24×24 CSS pixels**, or have spacing such that the combined target + spacing area meets 24×24px. Exceptions: inline text links; targets whose size is determined by the browser (e.g. native checkboxes, radio buttons).

Check with `browser_evaluate`:
```js
// Returns width and height of a specific element — both should be >= 24
document.querySelector('[aria-label="Close"]').getBoundingClientRect()
```

```css
/* ❌ Wrong — icon with no padding, renders below 24px */
.action-btn { padding: 0; width: 16px; height: 16px; }

/* ✅ Correct — ensure minimum 24×24px target area */
.action-btn {
  min-width: 24px;
  min-height: 24px;
  padding: 4px;
}
```

---

## 3.3.7 Redundant Entry — Re-asking for previously provided information

**WCAG:** 3.3.7 Redundant Entry | **Impact:** AA

In a multi-step process, information already entered by the user in the same session must not need to be re-entered unless re-entry is essential (e.g. confirming a password).

**Fix:** Pre-populate fields in later steps with values from earlier steps using session state, URL state, or a shared store:

```js
// Vanilla JS — read from session storage set in step 1
const step1Data = JSON.parse(sessionStorage.getItem('step1') ?? '{}')
document.getElementById('project-title').value = step1Data.title ?? ''
```

```tsx
// React — pass earlier step data as default values
function Step2({ step1Data }) {
  return <input id="title" defaultValue={step1Data.title} />
}
```

---

## 3.3.8 Accessible Authentication — Cognitive function test in auth flow

**WCAG:** 3.3.8 Accessible Authentication (Minimum) | **Impact:** AA

Authentication must not require the user to solve a cognitive puzzle (transcribe distorted text, solve a maths problem) as the only option. Acceptable alternatives include: copy-paste, password managers, object recognition (clicking images), link-based login, or WebAuthn.

**Fix:** Ensure login inputs are not blocked from password manager autofill:
```html
<!-- ❌ Wrong — prevents autofill -->
<input type="text" autocomplete="off" />

<!-- ✅ Correct — explicit autocomplete attribute -->
<input type="password" autocomplete="current-password" />
<input type="email" autocomplete="email" />
```

If using a CAPTCHA, ensure at least one alternative (audio CAPTCHA, email link, support contact) is provided.
