# Accessibility Remediation Plan — UQ Homepage

**Audited URL:** https://uq.edu.au/  
**Date:** 2026-05-21  
**Standard:** WCAG 2.2 AA  
**Tool:** Playwright MCP + axe-core  
**Tester:** Claude (Sonnet 4.6)  
**Environment:** Live — source code inaccessible

---

## Summary

| Severity | Count |
|---|---|
| Critical | 0 |
| High / Serious | 2 |
| Medium / Moderate | 0 |
| Low / Minor | 1 |
| **Total** | **3** |

All findings are open. This is a live URL audit — fixes require access to the UQ web team's source repository.

---

## Findings by Priority

### Priority 1 — High / Serious (fix first)

#### #1 — Footer link not distinguishable by color alone (WCAG 1.4.1)

| Field | Detail |
|---|---|
| **WCAG criterion** | 1.4.1 — Use of Color (Level A) |
| **Element** | `<a class="uq-footer__link">00025B</a>` (CRICOS registration link) |
| **Axe rule** | `link-in-text-block` |
| **Current state** | Link is white (`#ffffff`), surrounding text is light purple (`#d4c8de`). Contrast ratio: 1.6:1. No underline. |
| **Required** | Links in text blocks must either have a 3:1 contrast ratio vs surrounding text, OR use a non-color visual cue (e.g., underline). |
| **Recommended fix** | Add `text-decoration: underline` to `.uq-footer__link`, or restyle so the link color contrasts with surrounding footer text at ≥ 3:1. |
| **Effort** | Low — single CSS rule |

---

#### #2 — "Skip to footer" focus outline below minimum size (WCAG 2.4.11)

| Field | Detail |
|---|---|
| **WCAG criterion** | 2.4.11 — Focus Appearance (Level AA — new in WCAG 2.2) |
| **Element** | "Skip to footer" skip navigation link |
| **Axe rule** | Manual — keyboard check |
| **Current state** | Focus outline: 1px. Other skip links ("Skip to menu", "Skip to content") use a 3px outline and pass. |
| **Required** | Focus indicator must have a contrasting area ≥ 2px around the perimeter of the focused component. |
| **Recommended fix** | Apply the same focus style as the passing skip links — minimum 2px outline. Check whether the "Skip to footer" link is using a different CSS class or selector that misses the shared focus rule. |
| **Effort** | Low — CSS alignment, likely one selector missing from the shared focus rule |

---

### Priority 2 — Low / Minor (fix after high priority items)

#### #3 — Unlabelled `<nav>` landmark (ARIA Authoring Practices)

| Field | Detail |
|---|---|
| **WCAG criterion** | Best practice — supports WCAG 2.4.1 (Bypass Blocks) and 1.3.6 (Identify Purpose) |
| **Element** | One of 8 `<nav>` elements on the page |
| **Current state** | 7 nav elements have distinct `aria-label` attributes. 1 does not — announced as "navigation" with no context. |
| **Required** | Not a hard WCAG 2.2 AA failure, but ARIA best practice requires distinguishable labels when multiple instances of the same landmark type appear on a page. |
| **Recommended fix** | Identify the unlabelled nav element and add an `aria-label` that describes its purpose (e.g., `aria-label="Breadcrumb"`, `aria-label="Utility links"`, etc.). |
| **Effort** | Low — single HTML attribute |

---

## Recommended Fix Order

1. **#2 — Skip link focus outline** — 1-line CSS fix, zero risk
2. **#1 — Footer link contrast** — CSS rule; test visually to ensure the footer design is preserved
3. **#3 — Nav aria-label** — HTML attribute; identify the specific nav element first

---

## Testing After Fixes

- Re-run `/a11y-audit "uq.edu.au homepage" --code --keyboard --structure` to verify all three findings are resolved
- Confirm with NVDA or VoiceOver that the previously unlabelled nav is now announced with its label
- Verify the CRICOS link is identified as a link by screen readers after the contrast/underline fix

---

## Notes

- This audit covered the homepage only. Sub-pages, logged-in states, and interactive components (search results, modals) were not tested.
- The skip link finding (#2) and nav landmark finding (#3) are low-effort and should be bundled into a single ticket.
- The footer link finding (#1) may require coordination with the brand/design team if the purple-on-purple colour scheme is intentional.
