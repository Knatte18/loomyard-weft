---
round: 3
focus:
  - "Card 7's `Fabric.FastForwardPair` glyph and Intent: does a mutating fabric verb that moves the parent pair's warp through `ResetHard` and its weft through `PullWeft` have a signature and result that satisfy `PATTERN-mutation-record` (accumulates a `*Mutations` record, result embeds `MutationRecord`) and that supply the `SyncOptions` which `PullWeft` takes, with the refusal path covered by a named test?"
  - "Card 12's `parentPuller` and `syncParent` consumers of the fast-forward result: do they read the shape card 7 now plans, and does Merge-In pass the sync options card 7's signature needs?"
  - "Card 22's addition of a parent lock wait to the `lyx loom status` waiting key: is `contracts/specs/loom-status-spec.md`, whose `## The waiting key` section defines the waits, their read order and their render forms, edited by some card, and does the plan state where the lock wait sits relative to `gateWaiting`, the outermost wrapper in `reviewWaitingFor` in `internal/loomcli/reviewstatus.go`?"
---

Focus covers the two open findings and the fix surface they touch.
