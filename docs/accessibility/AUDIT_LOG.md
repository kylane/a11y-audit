# Accessibility Audit Log

## Audit Scope

Each audit covers the following dimensions:
- **Automated (WCAG 2.2 AA):** axe-core CDN injection via Playwright MCP
- **Keyboard:** Tab order, Escape-to-close, focus return, dialog behaviour
- **Structure:** Heading hierarchy, table markup, landmark regions
- **Forms:** Labels, required fields, validation messages, field grouping

---

## Audits

| Date | URL | Branch | Scope | Standard | Tool | Findings | Tester |
|------|-----|--------|-------|----------|------|----------|--------|
| 2026-05-21 | https://uq.edu.au/ | — | Homepage | WCAG 2.2 AA | Playwright MCP + axe-core | #1–#3 | Claude (Sonnet 4.6) |

---

## Detailed Findings

### #1 — HIGH: `link-in-text-block` — CRICOS footer link lacks contrast from surrounding text

- **Files:** n/a (live URL audit — source unavailable)
- **Axe rule:** `link-in-text-block` (WCAG 1.4.1 — Use of Color)
- **Impact:** Serious
- **Description:** The CRICOS registration number link (`00025B`) in the footer is rendered in white (`#ffffff`) on a purple background, surrounded by lighter purple text (`#d4c8de`). The contrast ratio between the link and surrounding text is 1.6:1 — well below the 3:1 minimum required for links that rely solely on color to distinguish them from surrounding text (no underline or other non-color cue is present). Users who cannot perceive color differences cannot identify this as a link.
- **Fix:** Not applied — live URL, source inaccessible. Remediation: add `text-decoration: underline` to the link, or increase the contrast ratio between link and surrounding text to at least 3:1.

---

### #2 — HIGH: "Skip to footer" focus outline below 2px minimum (WCAG 2.4.11)

- **Files:** n/a (live URL audit — source unavailable)
- **Axe rule:** Manual — keyboard check (WCAG 2.4.11 — Focus Appearance)
- **Impact:** Serious
- **Description:** The "Skip to footer" skip link renders a 1px focus outline when focused via keyboard. WCAG 2.4.11 (Level AA in WCAG 2.2) requires focus indicators to have a contrasting area of at least 2px around the perimeter of the focused component. The other two skip links ("Skip to menu", "Skip to content") display a 3px outline and pass. This inconsistency suggests the "Skip to footer" link is missing a focus style that matches the rest of the skip navigation.
- **Fix:** Not applied — live URL, source inaccessible. Remediation: apply the same focus style used by the other skip links (minimum 2px outline).

---

### #3 — LOW: One `<nav>` landmark has no `aria-label`

- **Files:** n/a (live URL audit — source unavailable)
- **Axe rule:** Manual — structure check (WCAG best practice — ARIA Authoring Practices)
- **Impact:** Minor
- **Description:** The page contains 8 `<nav>` landmark regions. Seven have distinct `aria-label` attributes allowing screen reader users to identify each navigation region (e.g., "Main", "Section", "Footer"). One nav element has no `aria-label`, so it is announced as "navigation" with no further context. Screen reader users browsing by landmark (common behaviour in NVDA, VoiceOver, JAWS) cannot distinguish this region from the others.
- **Fix:** Not applied — live URL, source inaccessible. Remediation: add an `aria-label` attribute to the unlabelled `<nav>` element that describes its purpose.

---

## Summary

**Total findings: 3**
- Critical: 0
- High/Serious: 2
- Medium/Moderate: 0
- Low/Minor: 1

**All findings resolved:** No — all 3 findings are open (live URL audit, source inaccessible)
