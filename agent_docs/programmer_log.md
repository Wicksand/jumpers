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

### Historical Primary Developer action

The Firebase setup and campaign permission decisions were subsequently provided and approved; the implementation below uses them.

## 2026-11-16 Firebase implementation

### Implemented

- Added Firebase modular browser SDK imports from the public Google CDN; no build step, paid service, service-account secret, or private key was added.
- Added Google sign-in/sign-out and visible authenticated-user state.
- Added campaign creation for signed-in users, with generated six-character short codes.
- Added signed-in player join-by-code and leave-campaign flows.
- Added Firestore persistence for a player's own sheet across all joined campaigns and restoration of the saved remote sheet.
- Added DM campaign member-sheet viewing. The DM sees sheet data but there is no edit path.
- Kept localStorage save/restore as a fallback when signed out or when remote saving fails.
- Fixed local dynamic-row restoration so it does not append duplicate blank rows.
- Added accessibility context to generated weapon/throwable inputs.

### Authorization file

- Added `firestore.rules`. It enforces: players read/write only their own sheet; DMs read member sheets but cannot write them; players cannot read other players' sheets; players may join multiple campaigns; players may delete their own membership; campaign short codes are checked against the campaign; and campaign ownership is tied to the authenticated creator.

### Checks

- `node --check` on the extracted module script -> PASS.
- Static feature/security assertions for doctype, viewport, Firebase SDK/auth/Firestore/campaign flows, local fallback, rules file, and absence of `innerHTML`, `document.write`, and `eval` -> PASS.
- HTML tag-balance check for `script` and `form` -> PASS (`1/1` each).
- `git diff --check` -> PASS with normal LF-to-CRLF working-copy warnings only.
- Firebase CLI availability check -> NOT FOUND; rules were not deployed or emulator-tested in this environment.
- Browser popup authentication, Firestore writes, rules deployment, and responsive/console checks are Unknown until a browser and Firebase project deployment/test session are available.

### Rules handoff

No derived RPG calculations were invented. Exact semantics are still required for Armor Class, Initiative, Speed, hit points, attributes, inventory capacity, skills/saving throws, weapons, throwables, and stratagem timing before those fields can be calculated or validated.

### Quality Checker handoff

Review `index.html` and `firestore.rules`. Test Google sign-in on the configured GitHub Pages domain, create a campaign, join from a second account, save a player sheet, verify the DM can read but not edit it, verify a player cannot read another player's sheet, and verify leaving removes that player's membership/sheet access. Firebase rules deployment/emulator testing is still required before treating authorization as verified.

## 2026-11-16 Authorization remediation

### Exact fix

- Added `isMember`, `isPlayerMember`, and membership-path helpers to `firestore.rules`.
- Sheet reads now require the target UID to have an active `player` membership in that campaign. A player can read only their own sheet; a DM can read only active player sheets while the DM's own DM membership and campaign ownership are still valid.
- Sheet create/update/delete now require active player membership and self ownership. Sheet ownership cannot be changed on update.
- Campaign documents now accept only the expected creation fields, require a six-to-eight-character uppercase alphanumeric invite code, and deny all direct updates/deletes. This makes `dmId` and `inviteCode` immutable and blocks privilege escalation through replacement fields.
- Invite-code documents are restricted to the expected `campaignId` field, can only be created by the campaign DM in the existing creation batch, and cannot be updated or deleted directly.
- Membership creation is restricted to the expected document ID, caller UID, role, and field set. Player membership requires a matching invite-code document and campaign code; DM membership requires the campaign's post-batch `dmId` to match the caller. Membership updates are denied, and only a player can delete their own membership.
- Changed the client leave flow in `index.html` to delete the player's sheet before deleting membership. This preserves the new active-membership requirement during cleanup.

### Checks run

- `git diff --check` -> PASS; only normal LF-to-CRLF working-copy warnings were emitted.
- Node version check -> PASS (`v26.13.1` available).
- Existing inline module JavaScript/static checks from the prior implementation remain applicable; no RPG calculations or field semantics were changed.
- Firebase CLI, Firestore Rules Emulator, and a browser remain unavailable in this environment, so rules compilation/deployment and live authorization tests are UNKNOWN.

### Required deployment and user steps

1. Deploy `firestore.rules` to the `heck-jumpers` Firebase project using Firebase CLI or paste the file into Firestore Database -> Rules, then publish it.
2. Test with two Google accounts: a non-member sheet read/write is denied; a player can read/write only their own active campaign sheet; a player cannot read another player's sheet; a DM can read a member sheet but cannot create/update/delete it; a player can leave and loses sheet access.
3. Confirm campaign creation still succeeds as one batch and joining still succeeds before the sheet write.
4. If Firebase reports a rules syntax or field-set error, stop deployment and record the exact error; do not loosen the rules to make the client pass.

No RPG rules were invented. Derived calculations remain blocked pending the rules document.
