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

| Date | Branch | Scope | Tool | Findings | Tester |
|------|--------|-------|------|----------|--------|
| [YYYY-MM-DD] | [branch] | [feature/page] | Playwright MCP + axe-core 4.10.3 | #1–#N | [name/Claude] |

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
### #7 — HIGH: `nested-interactive` violation in grants and ethics selection dialogs

- **Files:** `SelectGrantsDialog.tsx`, `SelectHumanEthicsDialog.tsx`, `SelectAnimalEthicsDialog.tsx`
- **Axe rule:** `nested-interactive` (WCAG 4.1.2 — Name, Role, Value)
- **Impact:** Serious
- **Description:** Items were built using `ListItemButton` (renders as `role="button"`) containing a `Checkbox` input. WCAG prohibits interactive controls nested inside other interactive controls. Even `aria-hidden="true"` + `tabIndex={-1}` on the checkbox does not satisfy axe — the `<input>` element inside a `[role="button"]` is flagged regardless.
- **Fix:** Replaced `ListItemButton + Checkbox` with `FormControlLabel + Checkbox` inside a plain `ListItem`. `FormControlLabel` wraps a single `<label>` element around the checkbox — one interactive element, no nesting. Applied across all three dialogs.
```

---

## Updating the Audit Table and Summary

After writing all findings, add a row to the Audits table:
```markdown
| 2026-05-15 | ethics-validation | Create record wizard + ethics/service dialogs | Playwright MCP + axe-core 4.10.3 | #7–#13 | Claude (Sonnet 4.6) |
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
