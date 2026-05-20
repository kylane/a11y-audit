# VoiceOver Accessibility Checks

Use this reference during Step 7 of the audit (`--voiceover` flag). macOS only.

VoiceOver is built into macOS — no installation needed. It works best with **Safari**; Chrome also works but with some differences in announcement phrasing.

---

## Setup

**Enable VoiceOver:** Cmd+F5 (or triple-press Touch ID on supported Macs)

**VO key:** Ctrl+Option (held together — abbreviated as VO throughout this document)

**Enable the Caption Panel** so VoiceOver speech appears as on-screen text:
1. Open VoiceOver Utility: VO+F8
2. Go to Visuals → Caption Panel
3. Enable "Show Caption Panel"

The Caption Panel is the macOS equivalent of NVDA Speech Viewer — it lets you read what VoiceOver is announcing without needing to listen to audio. Use it to copy announcements verbatim for the audit record.

---

## Key Commands

| Action | Command |
|---|---|
| Start/stop reading | VO+A |
| Read next item | VO+Right |
| Read previous item | VO+Left |
| Activate focused element | VO+Space |
| Open Rotor | VO+U |
| Next heading (in Rotor or quick nav) | VO+Cmd+H |
| Next form control | VO+Cmd+J |
| Next link | VO+Cmd+L |
| Next table | VO+Cmd+T |
| Interact with element/group | VO+Shift+Down |
| Stop interacting | VO+Shift+Up |
| Read current item | VO+F3 |

**Quick Navigation** (no VO key needed when enabled):
Press VO+Q to toggle Quick Navigation. With it on: Left/Right to read, H for next heading, B for next button, F for next form field.

---

## 1. Page and Landmark Announcement

When the page loads or focus enters the browser:

- [ ] **Page title announced** — VoiceOver reads the `<title>` element on load
- [ ] **Main landmark reachable** — open Rotor (VO+U), navigate to Landmarks, confirm `<main>` is listed
- [ ] **Navigation landmarks labelled** — multiple `<nav>` elements should have distinct names (e.g. "Primary navigation", "Breadcrumb navigation")
- [ ] **No orphaned content** — all meaningful content is within a landmark

### Reporting format

```
PAGE / LANDMARKS
✓ Page title announced: "Sign-up — MyApp"
✓ Main landmark present in Rotor
✗ Second <nav> has no aria-label — Rotor shows two items both named "navigation"
✗ Footer content not within any landmark
```

---

## 2. Heading Navigation

Open Rotor (VO+U) and navigate to Headings, or use VO+Cmd+H to move through headings.

- [ ] **All headings listed** — Rotor shows headings in the correct order
- [ ] **Heading levels make sense** — h1 at top, h2 for sections, h3 for subsections — no skipped levels
- [ ] **Heading text is descriptive** — each heading clearly identifies the section that follows
- [ ] **Single h1** — Rotor shows exactly one h1

### Reporting format

```
HEADINGS
✓ Single h1: "Create your account"
✓ All h2/h3 in Rotor, no skipped levels
✗ "Terms and conditions" section starts with h4 — h3 never appears
```

---

## 3. Interactive Element Names

Move through interactive elements with VO+Cmd+J (form controls) or VO+Cmd+L (links).

### Buttons
VoiceOver announces: `[Name], button` — or `[Name], dimmed, button` if disabled.

- [ ] Every button announces a meaningful name
- [ ] Icon-only buttons have `aria-label` — VoiceOver should not announce the icon's SVG title or an empty name
- [ ] Disabled buttons are announced as "dimmed" (not just invisible to the user)

### Links
VoiceOver announces: `[Name], link`

- [ ] No links announced as "link" with no name
- [ ] No links whose name is a URL (`https://...`) — use descriptive text or `aria-label`
- [ ] "Open in new tab" links announced — add `(opens in new tab)` to the label or use `aria-label`

### Select / Dropdown
VoiceOver announces: `[Label], popup button, [current value]`

- [ ] Label is announced (not just the current value)
- [ ] All options readable via VO+Shift+Down to interact, then VO+Down/Up

### Reporting format

```
INTERACTIVE ELEMENTS
✓ "Submit" button — announces "Submit, button"
✓ "Close" icon button — announces "Close, button"
✗ "Settings" icon button — announces "gear, button" (SVG title being read; needs aria-label)
✗ "Read more" link repeated 4 times — all announce "Read more, link" with no distinguishing context
```

---

## 4. Form Fields

Navigate to each form field with VO+Cmd+J.

VoiceOver announces a text input as: `[Label], text field` (and "required" if `required` or `aria-required="true"` is set).

- [ ] **Label announced** — field name is read before "text field" / "checkbox" / etc.
- [ ] **Required state announced** — required fields include "required" in the announcement
- [ ] **No placeholder-only labels** — placeholder text is not reliably announced; must have a `<label>` or `aria-label`
- [ ] **Hint text announced** — if `aria-describedby` links a hint, VoiceOver reads it after the label (may require VO+F3 to read full description)
- [ ] **Autocomplete not blocked** — password manager should be able to fill the field; `autocomplete` attribute is set correctly

### Trigger validation and observe

Submit the form with required fields empty, then navigate back to each field:

- [ ] **`aria-invalid="true"` announced** — VoiceOver says "invalid data" when focus returns to a field with an error
- [ ] **Error message announced** — linked via `aria-describedby`; reading the field with VO+F3 includes the error text
- [ ] **Error injected into live region** — if an error summary appears after submit, it should be announced via `role="alert"` or `aria-live="assertive"` without requiring focus
- [ ] **Focus moves on submit failure** — focus goes to the error summary or first invalid field

### Reporting format

```
FORM FIELDS
✓ "Email address" — announces "Email address, text field, required"
✓ Hint text linked via aria-describedby — announced on VO+F3
✗ "Date of birth" — announces "text field" with no label (placeholder only)
✗ Validation error for "Email" not announced after submit — not in aria-live region
✗ aria-invalid not set on "Email" after validation — VoiceOver says nothing about invalid state
```

---

## 5. Dialog / Modal Behaviour

Open a dialog and observe:

- [ ] **Dialog announced on open** — VoiceOver says `[Title], web dialog` (or similar)
- [ ] **Focus enters dialog** — keyboard focus is inside the dialog immediately after open
- [ ] **Dialog title announced** — the heading or `aria-labelledby` target is read
- [ ] **Content readable** — VO+Right navigates through dialog content in logical order
- [ ] **Escape closes dialog** — VoiceOver does not handle Escape natively; the site must close the dialog and move focus
- [ ] **Focus returns on close** — after dialog closes, focus returns to the element that triggered it
- [ ] **Background inert** — VO+U Rotor should not list elements outside the dialog while it is open (`aria-modal="true"` or `inert` on the background)

### Reporting format

```
DIALOG — "Add item"
✓ Announced as "Add item, web dialog" on open
✓ Focus enters dialog immediately
✗ Background not inert — Rotor lists links and buttons outside the dialog
✗ Escape does not close dialog — focus trapped with no way out via keyboard
```

---

## 6. Images

Navigate with VO+Cmd+G (next graphic) or check in Rotor > Images.

- [ ] **Meaningful images have alt text** — VoiceOver announces the alt attribute
- [ ] **Decorative images are skipped** — `alt=""` means VoiceOver does not announce the image at all
- [ ] **Complex images have descriptions** — charts, diagrams, or infographics have a long description via `aria-describedby` or adjacent text
- [ ] **No "image.png" or filename announcements** — images without alt attributes may announce their filename

### Reporting format

```
IMAGES
✓ Hero image — announces alt text "Team collaborating around a whiteboard"
✓ Decorative divider — skipped (alt="")
✗ Chart image — announces "chart.png" (missing alt attribute)
```

---

## 7. Table Navigation

Navigate with VO+Cmd+T (next table). Once inside, use VO+Shift+Down to interact, then VO+Right/Left to navigate cells.

- [ ] **Table announced with accessible name** — VoiceOver says `[Table name], table, N columns, N rows`
- [ ] **Column headers announced** — as you navigate cells, VoiceOver reads the column header before the cell value
- [ ] **Row headers announced** — if the table has row headers (`<th scope="row">`), they are read as context
- [ ] **No layout tables with data roles** — layout tables should have `role="presentation"` and no `<th>` elements

### Reporting format

```
TABLE — "Recent orders"
✓ Announced as "Recent orders, table, 4 columns, 10 rows"
✓ Column headers read before each cell
✗ Row header cells missing scope="row" — context not announced when navigating cells
```

---

## Step 7 Output Template (VoiceOver)

```
VOICEOVER AUDIT — [feature/page]
Browser: Safari [version] / macOS [version]
Caption Panel: [enabled / manual observation]

PAGE / LANDMARKS
[results]

HEADINGS
[results]

INTERACTIVE ELEMENTS
[results]

FORM FIELDS
[results]

DIALOGS
[results — or "No dialogs tested"]

IMAGES
[results]

TABLES
[results — or "No tables found"]

Summary: N issues found
```

---

## Common VoiceOver Announcement Patterns

| Element | Expected announcement |
|---|---|
| Text input | `[Label], text field` |
| Required text input | `[Label], required, text field` |
| Invalid text input | `[Label], invalid data, text field` |
| Checkbox (checked) | `[Label], checkbox, checked` |
| Checkbox (unchecked) | `[Label], checkbox, unchecked` |
| Select / dropdown | `[Label], popup button, [current value]` |
| Button | `[Label], button` |
| Disabled button | `[Label], dimmed, button` |
| Link | `[Label], link` |
| Heading h2 | `[Text], heading level 2` |
| Dialog on open | `[Title], web dialog` |
| Alert / live region | Announced immediately without focus |
| Image with alt | `[Alt text], image` |
| Decorative image | *(not announced)* |

---

## Known Differences from NVDA

| Behaviour | NVDA | VoiceOver |
|---|---|---|
| Select element | "combo box" or "list box" | "popup button" |
| Required fields | "required" in announcement | "required" in announcement (same) |
| aria-live regions | Announced promptly | Announced promptly in Safari; may lag in Chrome |
| Dialog name | Read from aria-labelledby | Read from aria-labelledby |
| Disabled elements | "unavailable" | "dimmed" |
| Tab key | Moves between interactive elements only | Same |
| Reading order | Follows DOM order | Follows DOM order |
| Caption/log | Speech Viewer (NVDA menu) | Caption Panel (VoiceOver Utility > Visuals) |
