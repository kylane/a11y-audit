# Project Configuration for a11y-audit

To customise the audit skill for your project, create an `a11y-config.md` file in the project root. The skill reads this file first, before auto-detection.

---

## Template

Copy and fill in only the values you need. Omit any field to use auto-detection.

```markdown
# a11y-config.md — Accessibility Audit Configuration

## Dev Server

url: http://localhost:3000

## Auth

instructions: |
  Log in as "uqtest" (super user). Navigate to http://localhost:3000 and use the
  sidebar to reach any page — never type route paths directly. Session expires after
  ~5 minutes of inactivity; return to home page to re-trigger auth.

## Screenshots

output_path: C:\Users\<username>\Pictures\a11y-screenshots\

## Audit Log

path: docs/ux/AUDIT_LOG.md

## Component Library

library: MUI (Material UI v9)
notes: |
  - Use FormControlLabel + Checkbox instead of ListItemButton + Checkbox for selectable lists
  - Use ListItem divider prop instead of Divider component="li"
  - Use sx={{ color: ... }} not color= prop on Typography for custom colours
  - HelpButton placed inside DialogTitle contaminates accessible name — move outside

## Lint

command: rtk nx lint web
notes: |
  Pre-existing lint issues (do not flag as new):
  - ServiceCollaboratorPermissions.tsx — setState-in-effect error
  - FormStepper.tsx — two `any` type warnings

## Navigation

notes: |
  Never guess or type route paths. Use the sidebar navigation.
  Check apps/web/src/routes.tsx only to verify a path — still navigate via the app UI.
  Standard user accounts (e.g. uqjdoe) crash on ethics, admin, and graduate school pages.

## Scope Notes

notes: |
  Focus on flows that involve dialogs and multi-step forms.
  The create-record wizard and ethics application wizard are the highest-priority flows.
```

---

## Field Reference

| Field | Description | Default if omitted |
|---|---|---|
| `url` | Dev server base URL | Auto-detected from config files, or `http://localhost:3000` |
| `auth.instructions` | How to authenticate | Ask the user |
| `screenshots.output_path` | Where to save screenshots (must be outside the repo) | OS-appropriate temp path |
| `audit_log.path` | Path to the audit log file | Auto-detected, or `docs/accessibility/AUDIT_LOG.md` |
| `component_library.library` | Primary component library name | Auto-detected from package.json |
| `component_library.notes` | Project-specific fix patterns | None |
| `lint.command` | Lint command to run after fixes | Auto-detected from package.json scripts |
| `lint.notes` | Pre-existing lint issues to ignore | None |
| `navigation.notes` | Project-specific navigation constraints | None |
| `scope.notes` | Which flows to prioritise | None |

---

## Notes for Skill Authors / Contributors

If the auto-detection in `SKILL.md` Step 1 doesn't cover a project's specific setup, the recommended fix is to document it in `a11y-config.md` rather than changing the skill itself. This keeps the skill generic and the project-specific knowledge in the project repo where it belongs.
