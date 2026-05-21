# Audit Log Format

Use this reference when recording findings in Step 10 of the audit.

---

## If an audit log already exists

Read the existing file to understand its structure and numbering. Append new findings using the same format — do not change the existing structure.

Find the highest existing finding number and continue from there.

Find the existing summary section and update the totals.

---

## If no audit log exists

Create `docs/accessibility/AUDIT_LOG.md` (or the path specified in `a11y-config.md`) using the template below.

---

## Audit Log Template (new file)

```markdown
# Accessibility Audit Log

## Audit Scope

Each audit covers the following dimensions:
- **Automated (WCAG 2.2 AA):** axe-core CDN injection via Playwright MCP
- **Keyboard:** Tab order, Escape-to-close, focus return, dialog behaviour
- **Screen reader:** NVDA manual checklist (printed per audit, results recorded here)

---

## Audits

| Date | URL | Branch | Scope | Standard | Tool | Findings | Tester |
|------|-----|--------|-------|----------|------|----------|--------|
| [YYYY-MM-DD] | [url] | [branch] | [feature/page] | WCAG 2.2 AA | Playwright MCP + axe-core 4.10.3 | #1–#N | [name/Claude] |

---

## Detailed Findings

[Findings go here — see format below]

---

## Summary

**Total findings: N**
- Critical: 0
- High/Serious: 0
- Medium/Moderate: 0
- Low/Minor: 0

**All findings resolved:** [Yes / No — list open items if No]
```

---

## Finding Entry Format

Each finding should be one numbered entry. Group systemic violations (same rule across multiple files) into a single entry.

```markdown
### #N — [SEVERITY]: [Short descriptive title]

- **Files:** `path/to/file.tsx`, `path/to/other.tsx`
- **Axe rule:** `rule-id` (WCAG N.N.N — Criterion Name)  
  *(or "Manual — keyboard check" / "Manual — NVDA" if not caught by axe)*
- **Impact:** Critical / Serious / Moderate / Minor
- **Description:** What the violation is, why it matters, and what a user with a disability experiences as a result.
- **Fix:** What was changed and why it resolves the violation. Include the pattern used if relevant.
```

**Severity mapping:**

| Axe impact | Severity label |
|---|---|
| critical | Critical |
| serious | High / Serious |
| moderate | Medium / Moderate |
| minor | Low / Minor |
| Manual keyboard | High / Serious (if blocking) or Low / Minor |
| Manual NVDA | High / Serious (if blocking) or Low / Minor |

---

## Example Finding

```markdown
### #7 — HIGH: `nested-interactive` violation in item selection dialogs

- **Files:** `ItemSelectionDialog.tsx`, `CategorySelectionDialog.tsx`
- **Axe rule:** `nested-interactive` (WCAG 4.1.2 — Name, Role, Value)
- **Impact:** Serious
- **Description:** Items were built using a clickable container component (renders as `role="button"`) containing a `<input type="checkbox">`. WCAG prohibits interactive controls nested inside other interactive controls. Even `aria-hidden="true"` + `tabIndex={-1}` on the checkbox does not satisfy axe — the `<input>` element inside a `[role="button"]` is flagged regardless.
- **Fix:** Replaced the clickable wrapper with a `<label>` element enclosing the checkbox — one interactive element, no nesting. Applied across both dialogs.
```

---

## Updating the Audit Table and Summary

After writing all findings, add a row to the Audits table:
```markdown
| 2026-05-15 | http://localhost:3000 | feature/item-selection | Item selection dialogs | WCAG 2.2 AA | Playwright MCP + axe-core 4.10.3 | #7–#13 | Claude (Sonnet 4.6) |
```

Update the Summary totals. Count each finding once even if it spans multiple files.

---

## Findings that were deferred or accepted

If a finding is identified but deliberately not fixed (e.g. too complex, third-party limitation, accepted risk), record it as:

```markdown
### #N — [SEVERITY]: [Title] *(deferred)*

- **Files:** ...
- **Axe rule:** ...
- **Impact:** ...
- **Description:** ...
- **Reason not fixed:** [Explain why — third-party component, excessive complexity, accepted risk, business decision]
- **Owner:** [Who is responsible for revisiting]
- **Review date:** [When to check again]
```

---

## CSV Export Format

Used by Step 15 when the `--report` flag is passed. Write one row per finding. Save the file to `SCREENSHOT_PATH` (default: `a11y-screenshots/` in the project root) as `a11y-<feature>-<YYYY-MM-DD>.csv`.

### Columns

| Column | Description | Example |
|---|---|---|
| `date` | Audit date (YYYY-MM-DD) | `2026-05-20` |
| `wcag_target` | WCAG version and level targeted | `2.2-AA` |
| `url` | URL that was audited | `http://localhost:3000` |
| `environment` | `local` (dev server, source modifiable) or `live` (remote URL, read-only) | `local` |
| `branch` | Git branch at time of audit. Leave blank when auditing a remote URL. | `feature/item-selection` |
| `feature` | Feature or page audited | `item selection dialog` |
| `finding` | Finding number | `#7` |
| `test_type` | Source of the finding | `static` / `axe` / `keyboard` / `nvda` / `structure` / `forms` |
| `rule_id` | Rule or check identifier — see table below | `nested-interactive` |
| `wcag_criterion` | WCAG success criterion number. Leave blank if not directly mappable. | `4.1.2` |
| `impact` | Severity of the finding | `critical` / `serious` / `moderate` / `minor` |
| `severity` | Human-readable severity label (maps from impact) | `High / Serious` |
| `cross_validated` | `true` if caught by both `--static` and `--code` (axe). Otherwise blank. | `true` |
| `files` | Affected source files, semicolon-separated. Blank for live URL audits where source is unavailable. | `ItemSelectionDialog.tsx;CategorySelectionDialog.tsx` |
| `description` | Short description of the violation | `Checkbox nested inside clickable container` |
| `status` | Resolution status | `fixed` / `deferred` / `open` |
| `notes` | Fix applied, reason deferred, or owner/review date for deferred findings | `Replaced container with label element wrapping checkbox` |

### `rule_id` values by test type

| `test_type` | `rule_id` format | Example |
|---|---|---|
| `static` | ESLint plugin rule ID | `jsx-a11y/label-has-associated-control` |
| `axe` | Axe-core rule ID | `nested-interactive` |
| `keyboard` | `keyboard-check` | `keyboard-check` |
| `nvda` | `nvda-check` | `nvda-check` |
| `voiceover` | `voiceover-check` | `voiceover-check` |
| `structure` | `heading-check` / `table-check` / `landmark-check` | `heading-check` |
| `forms` | `label-check` / `required-check` / `validation-check` / `grouping-check` | `validation-check` |

### Severity mapping

| Finding source | `impact` | `severity` |
|---|---|---|
| `axe` — critical | `critical` | Critical |
| `axe` — serious | `serious` | High / Serious |
| `axe` — moderate | `moderate` | Medium / Moderate |
| `axe` — minor | `minor` | Low / Minor |
| `static` | Use the ESLint rule severity (`error` → serious, `warn` → minor) | High / Serious or Low / Minor |
| `keyboard` | `serious` if focus is blocked or lost; `minor` otherwise | High / Serious or Low / Minor |
| `nvda` | `serious` if content is unannounced or misleading; `minor` otherwise | High / Serious or Low / Minor |
| `structure` | `serious` if heading hierarchy is broken or landmark is missing; `minor` for labelling issues | High / Serious or Low / Minor |
| `forms` | `serious` if error messages are unlinked or required fields unmarked; `minor` for grouping/autocomplete | High / Serious or Low / Minor |

### Example rows

```csv
date,wcag_target,url,environment,branch,feature,finding,test_type,rule_id,wcag_criterion,impact,severity,cross_validated,files,description,status,notes
2026-05-20,2.2-AA,http://localhost:3000,local,feature/item-selection,item selection dialog,#7,axe,nested-interactive,4.1.2,serious,High / Serious,true,ItemSelectionDialog.tsx;CategorySelectionDialog.tsx,Checkbox nested inside clickable container,fixed,Replaced container with label element wrapping checkbox
2026-05-20,2.2-AA,http://localhost:3000,local,feature/item-selection,item selection dialog,#7,static,jsx-a11y/no-interactive-element-to-noninteractive-role,4.1.2,serious,High / Serious,true,ItemSelectionDialog.tsx,Interactive element assigned non-interactive role,fixed,Same fix as axe finding — cross-validated
2026-05-20,2.2-AA,http://localhost:3000,local,feature/item-selection,item selection dialog,#8,keyboard,keyboard-check,2.1.1,serious,High / Serious,,ItemSelectionDialog.tsx,Escape key does not close dialog,fixed,Added onClose handler wired to Escape keydown
2026-05-20,2.2-AA,http://localhost:3000,local,feature/item-selection,item selection dialog,#9,nvda,nvda-check,,minor,Low / Minor,,ItemSelectionDialog.tsx,Dialog title not announced on open,deferred,Third-party component limitation — owner: UX team — review 2026-08-01
2026-05-20,2.2-AA,http://localhost:3000,local,feature/item-selection,item selection dialog,#10,structure,heading-check,1.3.1,serious,High / Serious,,ItemsPage.tsx,h3 appears before any h2 — skipped heading level,fixed,Corrected heading hierarchy
2026-05-20,2.2-AA,http://localhost:3000,local,feature/item-selection,item selection dialog,#11,forms,validation-check,3.3.1,serious,High / Serious,,ItemForm.tsx,Error message not linked via aria-describedby,fixed,Added aria-describedby linking field to error element
2026-05-20,2.2-AA,https://myapp.example,live,,item selection dialog,#12,axe,color-contrast,1.4.3,serious,High / Serious,,,Text contrast ratio 3.1:1 below 4.5:1 minimum,open,Deferred — source not accessible for live audit
```
