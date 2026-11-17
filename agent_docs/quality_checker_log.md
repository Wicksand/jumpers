# Quality Checker Review

Date: 2026-11-16

## Gate

**FAIL**

The implementation has a concrete Firestore authorization flaw, and live browser/Firebase verification could not be performed because neither a browser nor the Firebase CLI/emulator is available in this environment.

## Review scope

Read before testing:

- `agent_docs/quality_checker.md`
- `documents/task.md`
- `documents/coordinator.md`
- `agent_docs/firebase_setup.md`
- `agent_docs/Primary_developer.md`
- `agent_docs/programmer.md`
- `agent_docs/programmer_log.md`
- `index.html`
- `firestore.rules`
- `Helldivers Page 1.png`
- `Helldivers Page 2.png`

The working tree was inspected with `git status --short --branch`. Existing programmer changes were preserved.

## Checklist and results

| Area | Result | Evidence |
| --- | --- | --- |
| Static character-sheet page | Pass | `index.html` contains the editable form, local draft controls, dynamic weapon/throwable rows, print styling, and responsive CSS. |
| Firebase config wiring | Pass, live behavior Unknown | Firebase modular imports and config fields in `index.html` match `agent_docs/firebase_setup.md`; the API key is a browser key, not a service-account secret. Console restrictions cannot be verified from the repository. |
| Google authentication | Unknown | `signInWithPopup`, `signOut`, and `onAuthStateChanged` are wired, but no browser session was available to test the configured GitHub Pages domain. |
| Campaign creation/join/leave | Unknown | Client flows and short-code schema are present, but Firestore writes and rules were not deployed/emulator-tested here. |
| DM/player privacy | Fail | See the high-severity finding below. The intended read-only DM behavior is present in the client and partly represented in rules, but player sheet membership enforcement is missing. |
| RPG calculations | Pass with limitation | No calculations were invented; rules remain explicitly required before deriving values. |
| Reference layout | Pass with notes | Futuristic dark grid/control-board styling and major sheet sections are represented. The source is flatter than the reference: inspiration, proficiency bonus, saving throws, explicit skill rows, physical-characteristic fields, separate weapon groupings, and the stratagem table are not modeled as distinct controls. |
| Accessibility/static safety | Pass with notes | Labels, keyboard focus styles, live status text, table headers, and dynamic `aria-label`s exist. Browser contrast, keyboard, screen-reader, and responsive checks remain Unknown. No `innerHTML`, `document.write`, or `eval` usage was found. |
| GitHub Pages compatibility | Pass with notes | Static `index.html` plus public Firebase CDN imports require no build step or paid service. Live deployment/network checks are Unknown. |

## Findings

### High — sheet rules do not require campaign membership

Severity: `high`

Affected file/flow: `firestore.rules`, `campaigns/{campaignId}/sheets/{uid}` read/write/delete; player sheet access.

Reproduction:

1. Sign in as any Firebase user who is not a member of a known campaign.
2. Use the Firestore client or console to read `campaigns/<known-id>/sheets/<that-user-uid>` or write a document at that path with `ownerUid` equal to that UID.
3. The rule evaluates `isSelf(uid)` only; it never checks `memberships/<campaignId>_<uid>`.

Expected: a player can read/write/delete only their own sheet in a campaign they currently belong to.

Actual: any signed-in user who knows a campaign ID can read, create, update, or delete their own UID-named sheet in that campaign, whether or not they joined it. The same omission means self-sheet access is not revoked by membership deletion or campaign deletion.

Required action: add an active-membership check to sheet read/create/update/delete rules, while retaining the DM read-only exception. Deploy and test the corrected rules before approval.

### Medium — campaign ownership can be changed by an unrestricted DM update

Severity: `medium`

Affected file/flow: `firestore.rules`, `campaigns/{campaignId}` update.

Reproduction:

1. Authenticate as the current DM.
2. Update the campaign document and change `dmId` to another UID, or change `inviteCode` without synchronizing the invite-code document.

Expected: ownership and invite-code invariants remain controlled by explicit, validated operations.

Actual: `allow update, delete: if isDm(campaignId)` checks only the existing owner and imposes no constraints on the replacement document. A current DM can transfer ownership or create a campaign/invite-code mismatch through a direct client write. No UI currently exposes this, but Firestore rules must not rely on UI omission.

Required action: either make campaign ownership/invite-code fields immutable in rules or validate allowed updates explicitly. If ownership transfer is desired, define and test that workflow separately.

### Low — reference sections are flattened or omitted

Severity: `low`

Affected file/flow: character-sheet UI in `index.html`.

Expected: the interactive sheet closely preserves the supplied two-page structure.

Actual: the implementation provides broad equivalents, but uses general textareas for talents/skills, physical characteristics, and stratagems; it does not expose distinct inspiration, proficiency bonus, saving-throw, or primary/secondary weapon controls shown in the references.

Required action: rules/product clarification is needed before deciding which of these are functional fields and which are visual-only.

## Automated checks

Commands run:

- Extracted the module script from `index.html` and piped it to `node --check --input-type=module`: **PASS**.
- PowerShell structural checks for doctype, viewport, form/script balance, and Firebase presence: **PASS** (`1/1` form and `1/1` script).
- Static unsafe-sink scan for `innerHTML`, `document.write`, `eval(`, `serviceAccount`, `private_key`, and `secret`: **PASS**, no matches.
- `git diff --check`: **PASS**; only normal Git LF/CRLF working-copy warnings were emitted.
- `rg` inspection of `firestore.rules` authorization functions and match blocks: completed; findings above.
- Firebase CLI/browser executable discovery: **NOT FOUND** for `firebase`, Chrome/Edge/Firefox. Node was available.

## Unknown / user action required

- Firebase CLI deployment and Firestore Rules Emulator tests are **Unknown**. A user or developer with Firebase project access must deploy the corrected rules and run authorized/unauthorized tests.
- Google popup sign-in, authorized-domain behavior, Firestore persistence, campaign creation, short-code joining, multi-campaign membership, DM read-only viewing, and leave cleanup are **Unknown** until tested in a real browser at `https://wicksand.github.io/jumpers/` with two Google accounts.
- API-key restriction settings are **Unknown** from repository inspection; confirm the key is restricted to the intended web referrers and required Firebase APIs in Google Cloud Console.
- RPG rule details remain required before adding derived calculations or validation.

## Final recommendation

Do not deploy or accept the Firebase authorization model yet. Correct the membership check and campaign-field invariants, deploy the rules, then run the two-account browser and negative-path test matrix. After those results are recorded, perform a second Quality Checker gate.
