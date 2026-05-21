# Structure, Heading, and Table Checks

Use this reference during Step 8 of the audit (`--structure` flag).

Covers three areas: heading hierarchy, table markup, and landmark/region structure. Some violations are caught automatically by axe in Step 5 — this step adds the manual review layer that axe cannot assess.

---

## WCAG Criteria Covered

| Criterion | Description |
|---|---|
| 1.3.1 Info and Relationships | Structure conveyed visually must also be in the markup |
| 2.4.1 Bypass Blocks | Landmark regions allow keyboard users to skip repeated content |
| 2.4.6 Headings and Labels | Headings and labels must be descriptive |

---

## 1. Heading Hierarchy

### Automated (axe catches these)

| Axe rule | What it checks |
|---|---|
| `heading-order` | Heading levels must not skip (e.g. h1 → h3 with no h2) |
| `empty-heading` | Headings must not be empty |
| `p-as-heading` | Paragraphs styled to look like headings but not marked up as such |

### Manual checks

Run these in the browser using the Playwright snapshot or DevTools:

- [ ] **Single h1** — the page has exactly one `<h1>`, and it describes the page or feature
- [ ] **Logical order** — headings follow a strict hierarchy with no skipped levels
- [ ] **Descriptive** — each heading clearly describes the content that follows it; avoid generic headings like "Section" or "Details"
- [ ] **Not used for styling** — headings are not used purely to achieve a font size; visual styling should use CSS classes, not heading level changes
- [ ] **Dialog headings** — each dialog has an `<h2>` (or appropriate level) as its title, matching or used by `aria-labelledby`

### Reporting format

```
HEADING HIERARCHY
✓ Single h1: "Create Research Record"
✓ No skipped levels
✗ h3 "Filters" appears before any h2 — skipped level
✗ Dialog "Select category" has no heading — title is a <p> element
```

---

## 2. Table Markup

### Automated (axe catches these)

| Axe rule | What it checks |
|---|---|
| `scope-attr-valid` | `scope` attribute values must be valid (`col`, `row`, `colgroup`, `rowgroup`) |
| `td-headers-attr` | Cells using `headers` attribute must reference valid `<th>` `id` values |
| `th-has-data-cells` | Each `<th>` must have associated data cells |
| `table-duplicate-name` | Table `<caption>` and `summary` must not be identical |
| `table-fake-caption` | Data in the first row should use `<caption>`, not a merged cell |

### Manual checks

- [ ] **Data vs layout** — confirm whether each table conveys data or is used for layout; layout tables must have `role="presentation"` and no `<th>` elements
- [ ] **Column headers** — all column headers use `<th scope="col">`
- [ ] **Row headers** — all row headers use `<th scope="row">`
- [ ] **Complex tables** — tables with merged cells use `id`/`headers` association on every `<td>`
- [ ] **Accessible name** — every data table has an accessible name via `<caption>`, `aria-label`, or `aria-labelledby`
- [ ] **Empty cells** — empty `<th>` cells use `scope` and contain visually-hidden text if needed to convey meaning

### Reporting format

```
TABLE: "Collaborators" (line 142 CollaboratorsTable.tsx)
✓ Column headers use <th scope="col">
✓ Table has <caption>
✗ Row header cells missing scope="row"
✗ Merged cell at row 3 col 2 has no headers attribute
```

---

## 3. Landmark and Region Structure

### Automated (axe catches these)

| Axe rule | What it checks |
|---|---|
| `region` | All content must be within a landmark region |
| `landmark-one-main` | Page must have exactly one `<main>` landmark |
| `landmark-unique` | Landmarks of the same type must have unique accessible names |
| `landmark-complementary-is-top-level` | `<aside>` must not be nested inside `<main>` (unless intentional) |

### Manual checks

- [ ] **Main landmark** — page has exactly one `<main>` (or `role="main"`)
- [ ] **Navigation landmarks** — each `<nav>` has a unique `aria-label` when the page has more than one (e.g. "Primary navigation", "Breadcrumb")
- [ ] **All content in a landmark** — no meaningful content sits outside a landmark (`<header>`, `<main>`, `<footer>`, `<nav>`, `<aside>`, `<section aria-label="...">`)
- [ ] **Consistent across pages** — landmark structure matches between pages; users relying on screen readers build a mental model of the page layout
- [ ] **Skip link** — page has a visible-on-focus skip link that targets `<main>` (especially important for pages with large navigation headers)

### Reporting format

```
LANDMARK STRUCTURE
✓ Single <main> landmark
✓ Primary <nav> labelled "Primary navigation"
✗ Second <nav> has no aria-label — ambiguous for screen readers
✗ Footer content (copyright, links) sits outside any landmark
✗ No skip link found
```

---

## Step 8 Output Template

After running all checks, report findings in this format before moving to the fix step:

```
STRUCTURE AUDIT — [feature/page]

HEADINGS
[results]

TABLES
[results — list each table separately, or "No tables found"]

LANDMARKS
[results]

Summary: N issues found (N heading · N table · N landmark)
```

Carry any failures forward into Step 12 (Fix violations) alongside axe and keyboard failures.
