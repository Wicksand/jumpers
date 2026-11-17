# Primary Developer Communication

This file is the coordinator's communication channel for questions, approvals, and actions required from the primary developer.

## 2026-11-16

- Created `agent_docs/programmer.md` and `agent_docs/quality_checker.md` as the two role workflows requested by `documents/coordinator.md`.
- No user action is required yet.
- The project currently has an empty `index.html`; feature implementation has not been requested in this coordination step.

## Programmer questions / blockers — 2026-11-16

- Unknown: Firebase project ID, authorized domains, and Google sign-in configuration.
- Unknown: Firestore schema and security rules for campaign ownership, player membership, and DM access.
- Unknown: The RPG rules needed for derived values such as armor, initiative, hit points, and inventory capacity.
- Required action: provide or approve those Firebase and game-rule decisions before authentication, DM/player linking, or automated calculations are implemented.

## Approved campaign permissions - 2026-11-16

- DM can view player sheets: yes.
- DM can edit player sheets: no.
- Players can view other players' sheets: no.
- Players can join multiple campaigns: yes.
- Campaign invitation type: short code.
- Players can leave campaigns: yes.

Implementation implication: Firestore rules must allow a DM to read, but not write, member sheets; allow each player to read and write only their own sheet; deny players access to other members' sheets; support membership in multiple campaigns; and allow a player to remove their own membership. Campaign join codes must be stored and validated without exposing unrelated campaign data.

## RPG baseline approved - 2026-11-16

- Core statistics use standard D&D 5e conventions.
- Unless the workbook or a later decision specifies otherwise, use standard D&D 5e rules for calculations.
- Spells are replaced by stratagems and advanced-tech abilities.
- The source workbook is `Helldiver Campaign.xlsx` and includes Skill Trees, Soldier Class, Stratagems, Stratagem Comparisons, Weapons, Ship Upgrades, Planet Types, Weather, Surface Information, Ranks, Armor Benefits, and Lingering Injuries.
- Inventory capacity is measured in slots from Strength:
  - Strength 1-10: 30 slots.
  - Strength 11-15: 40 slots.
  - Strength 16-20: 50 slots.
- Each item has an assigned slot cost. The exact slot cost should come from the workbook when available; otherwise mark it as Unknown rather than inventing a value.
- The implementation may proceed with standard 5e-derived calculations, but custom mechanics that are not defined by 5e or the workbook must be recorded for later approval.

## Firebase implementation decisions - 2026-11-16

- Firestore schema: `campaigns/{campaignId}`, `inviteCodes/{shortCode}`, `memberships/{campaignId_uid}`, and `campaigns/{campaignId}/sheets/{uid}`.
- Campaign owner is stored as `dmId`; a DM receives read-only access to member sheets. The DM cannot write or delete a member sheet through the rules.
- A player membership is keyed by campaign and user, so one user can join multiple campaigns. A player can delete their own membership and own sheet when leaving.
- Invite-code lookup exposes only the campaign ID to signed-in users; the client then creates a membership only when the code matches the campaign. Firestore rules enforce the code/campaign relationship.
- Signed-in player saves write their own sheet to every joined campaign. Unsigned users retain localStorage drafts only.
- No RPG calculation was added. Exact rules still needed before changing or auto-calculating Armor Class, Initiative, Speed, hit point maximum, attributes, inventory capacity, skills, saving throws, weapon attack/damage, throwable use/damage, or stratagem timing fields.

## Quality Checker required actions - 2026-11-16

- Security issue: `firestore.rules` currently allows any signed-in user to read, create, update, or delete their own UID-named sheet under any known campaign path without proving membership in that campaign. The rules need a membership check for sheet access before deployment. This also leaves stale sheet access possible after a campaign document is deleted.
- Required Firebase action: deploy the corrected rules with the Firebase CLI or Firebase Console, then test the approved negative paths with two Google accounts: a non-member cannot access a campaign sheet, a player cannot access another player's sheet, and a DM can read but cannot write a member sheet.
- Required browser action: test Google sign-in on `https://wicksand.github.io/jumpers/`, campaign creation, short-code joining, multi-campaign membership, sheet save/restore, DM read-only viewing, and leaving a campaign. Browser and live Firebase testing are currently Unknown in this environment.

## Programmer authorization remediation - 2026-11-16

- Fixed `firestore.rules` so sheet access requires active membership in the addressed campaign. Players can read/write/delete only their own sheet while they are active `player` members; DMs can read active player sheets only while their DM membership and ownership remain valid, and cannot write or delete sheets.
- Campaign `dmId` and `inviteCode` are protected by denying direct campaign updates/deletes. Campaign creation, invite-code creation, and membership creation are field-limited and tied to the authenticated caller and the creation batch. Invite-code documents are immutable after creation.
- Membership role/owner fields cannot be edited. Only a player can delete their own membership. The client now deletes the player's sheet before membership when leaving, because the corrected sheet rule intentionally requires membership during deletion.
- No RPG calculations were added.

### Required action before acceptance

Deploy the current `firestore.rules` to Firebase and run the two-account negative-path matrix documented in `agent_docs/programmer_log.md`. Firebase rule compilation, deployment, and browser behavior remain `Unknown` until those user-side steps are performed. If deployment reports a syntax issue, provide the exact Firebase error before any rule relaxation.

## Rules implementation status - 2026-11-16

- Implemented in `index.html`: standard ability modifiers and Dexterity-based initiative.
- Implemented in `index.html`: inventory capacity display and item rows using the approved Strength bands.
- Armor class and hit-point maximum remain manual because armor configuration, class, level, and hit-die rules are not yet specified.
- Workbook content is available for a later structured data-import pass. Undefined item slot costs remain user-entered rather than guessed.
