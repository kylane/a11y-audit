# Project Configuration for a11y-audit

To customise the audit skill for your project, create an `a11y-config.md` file in the project root. The skill reads this file first, before auto-detection.

---

## Template

Copy and fill in only the values you need. Omit any field to use auto-detection.

```markdown
# a11y-config.md — Accessibility Audit Configuration

## Screenshots

output_path: C:\Users\yourname\Pictures\a11y-screenshots\

## Audit Log

path: docs/accessibility/AUDIT_LOG.md

## Component Library

library: [your component library name and version]
notes: |
  - Any project-specific fix patterns or known component quirks go here
  - For example: which component to use instead of a known inaccessible pattern

## Lint

command: npm run lint
notes: |
  Pre-existing lint issues to ignore (do not flag as new):
  - List any known pre-existing violations here so they aren't reported as new findings

## Navigation

notes: |
  Any project-specific navigation constraints — e.g. which test account to use,
  whether certain routes require specific permissions, or areas to avoid.

## Scope Notes

notes: |
  Which flows or pages to prioritise in this audit.

## WCAG Target

wcag: 2.2-AA
```

---

## Field Reference

| Field | Description | Default if omitted |
|---|---|---|
| `screenshots.output_path` | Where to save screenshots (must be outside the repo) | OS-appropriate temp path |
| `audit_log.path` | Path to the audit log file | Auto-detected, or `docs/accessibility/AUDIT_LOG.md` |
| `component_library.library` | Primary component library name | Auto-detected from package.json |
| `component_library.notes` | Project-specific fix patterns | None |
| `lint.command` | Lint command to run after fixes | Auto-detected from package.json scripts |
| `lint.notes` | Pre-existing lint issues to ignore | None |
| `navigation.notes` | Project-specific navigation constraints | None |
| `scope.notes` | Which flows to prioritise | None |
| `wcag.wcag` | Default WCAG version and level for all audits in this project | `2.2-AA` |

---

## Notes for Skill Authors / Contributors

If the auto-detection in `SKILL.md` Step 1 doesn't cover a project's specific setup, the recommended fix is to document it in `a11y-config.md` rather than changing the skill itself. This keeps the skill generic and the project-specific knowledge in the project repo where it belongs.
