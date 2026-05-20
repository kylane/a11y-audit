# Accessibility Fix Patterns

Patterns organised by axe rule ID, with framework-specific examples where relevant.
Load this file during the fix phase of an a11y audit.

**Target standard: WCAG 2.2 AA**

> **Note — WCAG 2.2 change:** Success criterion 4.1.1 Parsing has been removed from WCAG 2.2. Axe may still report `duplicate-id` and similar parsing rules as informational, but they are no longer a conformance requirement under WCAG 2.2. They remain worth fixing for robustness but should not be counted as AA failures.

---

## `nested-interactive` — Interactive control inside another interactive control

**WCAG:** 4.1.2 Name, Role, Value | **Impact:** Serious

Caused by placing a checkbox, button, or input inside a clickable container (button, link, or element with `role="button"`). Assistive technologies cannot reliably interact with both.

**MUI (React):**
```tsx
// ❌ Wrong — Checkbox inside ListItemButton (role="button")
<ListItemButton onClick={() => toggle(item)}>
  <Checkbox checked={isSelected(item)} tabIndex={-1} aria-hidden="true" />
  <ListItemText primary={item.label} />
</ListItemButton>

// ✅ Correct — FormControlLabel with single interactive control
<ListItem disablePadding divider={index < items.length - 1}>
  <FormControlLabel
    sx={{ width: '100%', mx: 0, px: 2, py: 0.5 }}
    control={<Checkbox edge="start" checked={isSelected(item)} onChange={() => toggle(item)} />}
    label={<ListItemText primary={item.id} secondary={item.label} />}
  />
</ListItem>
```

**Generic HTML:**
```html
<!-- ❌ Wrong -->
<div role="button" onclick="toggle()">
  <input type="checkbox" aria-hidden="true" tabindex="-1" />
  <span>Label</span>
</div>

<!-- ✅ Correct -->
<label>
  <input type="checkbox" onchange="toggle()" />
  Label text
</label>
```

**Radix UI / Headless UI:**
Use `Checkbox` root with a `label` element wrapping the whole thing, or use the library's `CheckboxItem` inside a proper list — do not nest inside `Button`.

---

## `list` — Invalid direct children of list element

**WCAG:** 1.3.1 Info and Relationships | **Impact:** Moderate

`<ul>` and `<ol>` must only contain `<li>` elements (or elements with `role="listitem"`). Dividers, wrappers, or elements with `role="presentation"` as direct children fail this rule.

**MUI (React):**
```tsx
// ❌ Wrong — Divider renders <li role="presentation"> inside <ul>
<List>
  <ListItem>Item 1</ListItem>
  <Divider component="li" />
  <ListItem>Item 2</ListItem>
</List>

// ✅ Correct — divider prop applies CSS border, no extra DOM element
<List>
  {items.map((item, index) => (
    <ListItem key={item.id} disablePadding divider={index < items.length - 1}>
      ...
    </ListItem>
  ))}
</List>
```

**Generic HTML:**
```html
<!-- ❌ Wrong -->
<ul>
  <li>Item 1</li>
  <hr role="presentation" />
  <li>Item 2</li>
</ul>

<!-- ✅ Correct — use CSS border on li instead -->
<ul>
  <li style="border-bottom: 1px solid #ccc;">Item 1</li>
  <li>Item 2</li>
</ul>
```

---

## `dialog-name` — Dialog has no accessible name

**WCAG:** 4.1.2 Name, Role, Value | **Impact:** Serious

Every `role="dialog"` must have an accessible name via `aria-labelledby` (preferred) or `aria-label`.

**Common cause 1 — Missing `aria-labelledby`:**
```tsx
// ❌ Wrong
<Dialog open={true}>
  <DialogTitle>Add item</DialogTitle>
</Dialog>

// ✅ Correct
<Dialog open={true} aria-labelledby="dialogtitle">
  <DialogTitle id="dialogtitle">Add item</DialogTitle>
</Dialog>
```

**Common cause 2 — HelpButton inside DialogTitle contaminates accessible name:**

When a `HelpButton` (with its own `aria-label`) is a child of `<DialogTitle>`, ARIA's accessible name computation traverses children and concatenates the HelpButton's label onto the dialog title.

```tsx
// ❌ Wrong — dialog announces "Add item Help" or similar
<DialogTitle id="dialogtitle">
  Add item
  <HelpButton aria-label="Help" />
</DialogTitle>

// ✅ Correct — move HelpButton outside DialogTitle
<Box sx={{ display: 'flex', alignItems: 'center', pr: 1 }}>
  <DialogTitle id="dialogtitle" sx={{ flex: 1 }}>Add item</DialogTitle>
  <HelpButton aria-label="Help" />
</Box>
```

---

## `dialog-name` + missing `onClose` — Dialog not keyboard-dismissible

**WCAG:** 2.1.2 No Keyboard Trap | **Impact:** Serious

A dialog without an `onClose` handler ignores both Escape key and backdrop click. Keyboard users cannot dismiss it without using an explicit close/cancel button — which may not exist in all states.

**MUI (React):**
```tsx
// ❌ Wrong — Escape key does nothing
<Dialog open={true}>

// ✅ Correct
<Dialog open={true} onClose={props.onClose}>
```

**Backdrop click on data-entry dialogs:**

For dialogs with significant form input, disable backdrop click to prevent accidental data loss while keeping Escape working:

```tsx
<Dialog
  open={true}
  onClose={(_, reason) => {
    if (reason === 'backdropClick') return
    props.onClose()
  }}
>
```

Use judgment: lightweight selection dialogs (a few checkboxes) can allow backdrop click. Heavy form dialogs (multiple fields, file uploads) should disable it.

---

## `label` — Form field has no accessible label, or label is excessively long

**WCAG:** 1.3.1, 2.4.6 | **Impact:** Critical / Moderate

**Missing label:**
```tsx
// ❌ Wrong
<input type="text" placeholder="Search..." />

// ✅ Correct — explicit label
<label htmlFor="search">Search</label>
<input id="search" type="text" placeholder="Search..." />

// ✅ Correct — aria-label when visible label not possible
<input type="text" aria-label="Search" placeholder="Search..." />
```

**Label too long (sentence used as field name):**

Screen readers announce the label on every focus. A 100+ character sentence is disorienting.

```tsx
// ❌ Wrong — full sentence as label
<TextField label="The lead investigator of a research project, or the Principal Advisor of an HDR project, who must approve the record (cannot be an HDR student)" />

// ✅ Correct — short label, description in helperText (linked via aria-describedby)
<TextField
  label="Lead Investigator"
  helperText="The lead investigator of a research project, or the Principal Advisor of an HDR project..."
/>
```

---

## `scrollable-region-focusable` — Scrollable region not keyboard-reachable

**WCAG:** 2.1.1 Keyboard | **Impact:** Serious

A scrollable container that is not focusable cannot be scrolled by keyboard-only users.

```html
<!-- ❌ Wrong -->
<div style="overflow-y: scroll; max-height: 300px;">...</div>

<!-- ✅ Correct -->
<div style="overflow-y: scroll; max-height: 300px;" tabindex="0" role="region" aria-label="Results list">...</div>
```

In React/MUI, add `tabIndex={0}` and an appropriate `aria-label` to the scrollable `<Box>` or `<div>`.

---

## `color-contrast` — Insufficient colour contrast

**WCAG:** 1.4.3 Contrast (Minimum) | **Impact:** Serious

Normal text requires 4.5:1 contrast ratio. Large text (18pt / 14pt bold) requires 3:1.

**Do not use raw hex values — use theme tokens** (they are pre-validated for contrast):
```tsx
// ❌ Wrong — raw hex, contrast unverified
<Typography sx={{ color: '#aaa' }}>Hint text</Typography>

// ✅ Correct — theme token (verified against background)
<Typography color="text.secondary">Hint text</Typography>
```

To check a custom colour: use the [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/) or the browser DevTools accessibility panel.

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

In React: `<img alt="" />` (empty string, not missing) for decorative images.

---

## `button-name` — Button has no accessible name

**WCAG:** 4.1.2 | **Impact:** Critical

```tsx
// ❌ Wrong — icon button with no label
<IconButton onClick={onClose}>
  <CloseIcon />
</IconButton>

// ✅ Correct — aria-label or Tooltip wrapping
<Tooltip title="Close">
  <IconButton onClick={onClose} aria-label="Close">
    <CloseIcon />
  </IconButton>
</Tooltip>
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

// ✅ Option B — don't hide it, just remove from tab order
<div tabIndex={-1}>Content for programmatic focus only</div>
```

---

## `region` — Content not within a landmark

**WCAG:** 1.3.6 | **Impact:** Moderate

Major content areas should be wrapped in landmark elements to aid screen reader navigation.

```html
<!-- ✅ Use semantic landmarks -->
<header> / <nav> / <main> / <section aria-label="..."> / <footer>

<!-- ✅ Or ARIA roles -->
<div role="main"> / <div role="navigation" aria-label="Primary">
```

---

## `scope-attr-valid` / empty table header — Column header has no accessible name

**WCAG:** 1.3.1 | **Impact:** Moderate

Table header cells (`<th>`) must have visible text or an `aria-label`. An empty `<th>` (e.g. containing only an icon button) leaves the column without context.

```tsx
// ❌ Wrong — empty header cell
<TableCell><HelpIconButton /></TableCell>

// ✅ Correct — explicit label
<TableCell aria-label="Help"><HelpIconButton /></TableCell>
```

---

## `tabindex` — Positive tabindex breaks natural tab order

**WCAG:** 2.4.3 Focus Order | **Impact:** Moderate

Positive `tabIndex` values (1, 2, 3...) override the natural DOM order and confuse keyboard users. Only ever use `tabIndex={0}` (in tab order, natural position) or `tabIndex={-1}` (programmatic focus only, not in tab order).

```tsx
// ❌ Wrong
<button tabIndex={3}>Submit</button>

// ✅ Correct — rely on DOM order
<button>Submit</button>

// ✅ Correct — programmatic focus target not in tab order
<div ref={skipToRef} tabIndex={-1} className="sr-only" aria-live="polite">
```

---

## `aria-live` — Dynamic content not announced

Not an axe rule — caught during keyboard/SR checks.

Dynamic content that updates without a page reload (validation errors, step changes, notifications) must be announced to screen reader users via a live region.

```tsx
// ✅ Polite — waits for user to finish current interaction
<div aria-live="polite" aria-atomic="true">
  {statusMessage}
</div>

// ✅ Assertive — interrupts immediately (use sparingly, for critical errors only)
<div aria-live="assertive">
  {criticalError}
</div>
```

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

**MUI:** MUI's default focus-visible styles generally meet 2.4.11 when using theme colours. Custom components that override `:focus` or use `outline: none` need explicit remediation.

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

```tsx
// ❌ Wrong — drag-to-reorder with no alternative
<Draggable onDragEnd={reorder}>...</Draggable>

// ✅ Correct — add keyboard/click-based reorder controls
<Draggable onDragEnd={reorder}>
  <IconButton aria-label="Move up" onClick={() => moveUp(index)}><ArrowUpIcon /></IconButton>
  <IconButton aria-label="Move down" onClick={() => moveDown(index)}><ArrowDownIcon /></IconButton>
  ...
</Draggable>
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

**MUI fix:** MUI `IconButton` defaults to 40px — no issue. Custom small buttons or icon-only links may need `minWidth`/`minHeight`:

```tsx
// ❌ Wrong — 16px icon with no padding
<Box component="button" sx={{ p: 0 }}><CloseIcon fontSize="small" /></Box>

// ✅ Correct — ensure minimum 24px target
<IconButton size="small" aria-label="Close">
  <CloseIcon fontSize="small" />
</IconButton>
// MUI IconButton size="small" renders at 34px — meets requirement
```

---

## 3.3.7 Redundant Entry — Re-asking for previously provided information

**WCAG:** 3.3.7 Redundant Entry | **Impact:** AA

In a multi-step process, information already entered by the user in the same session must not need to be re-entered unless:
- Re-entry is essential (e.g. confirming a password)
- The info was provided in a different context where re-use would be inappropriate

**Fix:** Pre-populate fields in later steps with values from earlier steps. Use form state management (React Hook Form `defaultValues`, context, or session storage) to carry values forward.

```tsx
// ✅ Pre-populate from earlier step data
const { data: recordData } = useGetDraftRecord(recordId)
<TextField
  label="Project title"
  defaultValue={recordData?.title}  // pre-filled from step 1
/>
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
