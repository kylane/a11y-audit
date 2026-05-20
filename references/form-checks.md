# Form Accessibility Checks

Use this reference during Step 8 of the audit (`--forms` flag).

Covers four areas: labels and accessible names, required field indication, validation and error messages, and field grouping. Some violations are caught automatically by axe in Step 4 — this step adds the manual layer axe cannot assess, particularly around dynamic validation behaviour.

---

## WCAG Criteria Covered

| Criterion | Description |
|---|---|
| 1.3.1 Info and Relationships | Form structure and relationships conveyed programmatically |
| 1.3.5 Identify Input Purpose | Personal data fields carry correct `autocomplete` attributes |
| 3.3.1 Error Identification | Errors identified in text and described to the user |
| 3.3.2 Labels or Instructions | Every input has a label or sufficient instructions |
| 3.3.3 Error Suggestion (AA) | Where known, the fix is suggested in the error message |
| 3.3.4 Error Prevention (AA) | Legal/financial/data submissions can be reviewed or reversed |

---

## 1. Labels and Accessible Names

### Automated (axe catches these)

| Axe rule | What it checks |
|---|---|
| `label` | Every form element must have an associated label |
| `label-content-name-mismatch` | Visible label text must match or be contained in the accessible name |
| `select-name` | `<select>` elements must have an accessible name |
| `input-button-name` | `<input type="button">` / `<input type="submit">` must have an accessible name |
| `autocomplete-valid` | `autocomplete` attribute values must be valid tokens |

### Manual checks

- [ ] **No placeholder-only labels** — `placeholder` disappears on input; it must not be the sole label for a field
- [ ] **Label association method** — each label uses one of: `<label for="id">`, `aria-label`, or `aria-labelledby` (not implicit wrapping alone for complex components)
- [ ] **Visible label matches accessible name** — what the user reads is what assistive technology announces; avoid hidden labels that differ from the visible text
- [ ] **Autocomplete on personal data fields** — inputs collecting name, email, address, phone, and payment details carry the correct `autocomplete` token (e.g. `given-name`, `email`, `street-address`, `tel`, `cc-number`)

### Reporting format

```
LABELS
✓ All inputs have associated labels
✗ "Date of birth" uses placeholder only — no <label> or aria-label
✗ Email input visible label is "Email" but aria-label is "Enter your email address here" — mismatch
✗ Name field missing autocomplete="given-name"
```

---

## 2. Required Fields

### Manual checks (axe does not reliably catch these)

- [ ] **Machine-readable required** — required fields carry the HTML `required` attribute or `aria-required="true"`; screen readers announce "required" when the field receives focus
- [ ] **Visual indicator explained** — if an asterisk or other symbol marks required fields, the legend ("* required") appears before the first required field in the reading order
- [ ] **Not colour alone** — required status is not communicated solely by colour (WCAG 1.4.1)
- [ ] **Optional fields labelled (where applicable)** — in forms where most fields are required, labelling optional fields rather than required ones reduces cognitive load and is a best practice

### Trigger and observe

Submit the form with all required fields empty, then verify:

- [ ] Focus moves to the first error or an error summary
- [ ] Each unfilled required field is identified as an error
- [ ] The error message for a required field uses plain language: "First name is required" not just "Required" or "Invalid"

### Reporting format

```
REQUIRED FIELDS
✓ All required fields have required attribute
✗ "Organisation" field has no required attribute — announced as optional
✗ Asterisk legend "* required" appears after the first required field
✗ Required fields distinguished by red border only — no text alternative
```

---

## 3. Validation and Error Messages

This is the highest-impact area for keyboard and screen reader users. Test by triggering validation errors.

### Automated (axe catches these partially)

| Axe rule | What it checks |
|---|---|
| `aria-live` regions | Elements with `aria-live` must be in the DOM before content is injected |

### Manual checks — error announcement

- [ ] **`aria-invalid="true"`** — set on each field that fails validation; removed when the error is corrected
- [ ] **Error linked via `aria-describedby`** — the error message element's `id` is in the field's `aria-describedby`; a screen reader announces the error message when the field receives focus
- [ ] **Dynamic errors announced** — if errors are injected into the DOM after submission, they are in an `aria-live="polite"` region or `role="alert"` so they are announced without requiring focus
- [ ] **Error message content** — each message names the field and describes what went wrong and (where possible) how to fix it: "Email address must include an @ symbol" not "Invalid email"
- [ ] **Errors persist** — error messages remain visible while the user corrects the field; they do not time out or disappear on focus

### Manual checks — focus management on submit

- [ ] **Focus on error summary** — if a summary list is used, focus moves to it on submission failure; the summary links each error to its field
- [ ] **Focus on first error** — if no summary is used, focus moves to the first invalid field
- [ ] **No silent failure** — submitting an invalid form does not simply reset or do nothing without any indication to the user

### Manual checks — inline (real-time) validation

- [ ] **Validate on blur, not on keystroke** — errors appear after the user leaves a field, not while they are typing; keystroke validation causes excessive announcements
- [ ] **Success not announced prematurely** — do not announce success (e.g. checkmark) while the user is still typing

### Trigger sequence

1. Navigate to the form via the app UI
2. Submit without filling in any fields — observe focus and announcements
3. Tab through each field with an error — confirm `aria-invalid` and `aria-describedby` are present
4. Fill in one field correctly — confirm `aria-invalid` is removed and the error message disappears
5. Check DevTools to verify `aria-describedby` values reference real element `id`s

### Reporting format

```
VALIDATION — "Contact" form
✓ aria-invalid="true" set on all invalid fields after submission
✓ Focus moves to error summary on submit
✗ Error message for "Email" not linked via aria-describedby (id mismatch: "email-error" vs "emailError")
✗ "Date" field error injected into DOM but not in an aria-live region — not announced
✗ Error message reads "Invalid date" — does not describe the expected format
✗ Errors disappear after 5 seconds
```

---

## 4. Field Grouping

### Automated (axe catches these)

| Axe rule | What it checks |
|---|---|
| `radiogroup` | Radio groups must be within a group role with accessible name |

### Manual checks

- [ ] **Radio groups in `<fieldset>`** — each group of radio buttons is wrapped in `<fieldset>` with a `<legend>` that states the question (e.g. "Preferred contact method")
- [ ] **Checkbox groups in `<fieldset>`** — same pattern for groups of checkboxes representing a multi-select question
- [ ] **`<legend>` is the first child** — `<legend>` must be the first element inside `<fieldset>` to be reliably announced
- [ ] **Single checkbox does not need `<fieldset>`** — a lone checkbox (e.g. "I agree to the terms") only needs a `<label>`, not a fieldset

### Reporting format

```
FIELD GROUPING
✓ "Notification preferences" checkbox group uses <fieldset> + <legend>
✗ "Study type" radio group has no <fieldset> — options announced without context
✗ <legend> "Contact details" is not the first child of its <fieldset>
```

---

## Step 8 Output Template

After running all checks, report findings before moving to the fix step:

```
FORM AUDIT — [feature/page] — [form name if multiple forms on page]

LABELS
[results]

REQUIRED FIELDS
[results]

VALIDATION
[results]

FIELD GROUPING
[results]

Summary: N issues found (N label · N required · N validation · N grouping)
```

Carry any failures forward into Step 9 (Fix violations) alongside findings from other test steps.
