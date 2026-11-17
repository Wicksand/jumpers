# Programmer Handoff

## 2026-11-16

### Scope implemented

Implemented the first usable Heck Jumpers character-sheet scope in `index.html`, based on the supplied HellDivers reference pages and the task brief:

- Futuristic dark control-board layout with responsive wide and narrow viewport styles.
- Accessible labeled fields for identity, attributes, combat readiness, vitals, inventory, physical characteristics, talents, weapons, throwables, and stratagems.
- Add/remove controls for weapon and throwable rows.
- Local-browser draft save/restore/clear using `localStorage`.
- Print-friendly stylesheet and print action.
- Visible status messaging and keyboard focus styling.
- No Firebase credentials, secrets, paid services, or network persistence added.

### Checks run

- `node --check "$env:TEMP\heck-jumpers-script.js"` — PASS; the inline JavaScript extracted from `index.html` parses successfully.
- PowerShell static checks for doctype, viewport, form, save control, responsive CSS, local storage, and absence of Firebase secret-shaped values — PASS.
- Structural tag count check for `<form>`/`</form>` and `<script>`/`</script>` — PASS.
- Browser console/manual viewport check — UNKNOWN; no supported browser executable was available on PATH in this environment.

### Known limitations / unresolved requirements

- Firebase Google authentication and DM/player campaign linking are not implemented. The task brief does not provide a Firebase project, authorized-domain configuration, schema, or campaign permission rules. Adding client-side credentials or inventing those rules would be unsafe.
- The sheet uses local browser storage only, so drafts do not sync between devices or users.
- Exact RPG calculations and field semantics are Unknown because no rules document was provided.

### Quality Checker handoff

Open `index.html` directly or serve the project directory with any static HTTP server. Verify the sheet at a desktop viewport and a narrow mobile viewport; enter a character name, add/remove rows, save, reload, clear, print, and check the browser console. Report the browser check as Unknown if no browser is available.

### Required Primary Developer action

Before the authentication/linking scope can be implemented, provide or approve the Firebase project setup, authorized domains, authentication providers, Firestore data model, and DM/player authorization rules.
