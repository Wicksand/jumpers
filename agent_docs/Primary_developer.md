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
