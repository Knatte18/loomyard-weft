# Fixer report — webster round 1

## Changes

- F1: `PATTERN.md:49` carries the declared "told executable-key lookup"; "Giving a recipe row a config value" became "a value" to fit the 320-character cap (315 now).
- F2: `foreignFlagRequest` sets `req.WeftPath` to the records worktree for a code-side flag.
  Fixed differently inside the test: the respawn rows of `TestCoalesceFlagged_PendingFlags` now expect `WeftPath` on a code-side request; the proposed run through `CoalescePushRequest` was not added, since the spawned request is what the defect corrupts and the existing rows fail without the production line.
- F3: fixed differently.
  The proposed change to `statusPushRequest` was not made: `PushAnchored`, `PushPairAnchored` and `Commit` share it and rely on its warp side surviving a marked records branch.
  `SpawnStatusPush` spawns nothing when `WeftPath` is empty instead (the review's alternative), with a new `TestSpawnStatusPush` row for a pair whose records branch alone is marked.
- F4: the final adoption loop in `add.go` re-marks each already cleared branch before `rollbackAdd`; the comment above it and the twin in `doc.go` are corrected.
- F5: kept; the `PushAnchored` doc comment now says it is the waiting records-only push for callers outside the Fabric Vocabulary owner set.
- F6: `spawn.go` header, `PushWarpAt` doc and `CoalescePushBothAt` doc no longer name it as the child's push.
- F7: test comment says "Command rows".

## Self-check

- fix-code-correctness: no findings.
- fix-affected-set: no findings.
- fix-consistency, LOW: `doc.go` kept the "never has a mark to restore" premise — fixed as proposed.
- fix-consistency, LOW: `doc.go` omitted the no-records-side spawn rule — fixed as proposed (plus the `TestSpawnStatusPush` doc comment).
- fix-consistency, LOW: `CoalescePushBothAt` mentions in `doc.go` — not a defect, no change.
- fix-proposal, LOW: `CoalescePushBothAt` doc named another symbol — fixed with a standalone sentence.
- fix-proposal, LOW: no test pins the F4 re-mark; the failing second clear cannot be injected without a seam, and the review asked for no test — not added.
- fix-proposal, LOW: ignored re-mark errors in F4 — rare best-effort restore inside a rollback; no change.

## Disputed

None.

## Deferred

None.

## Verification

`lyx gate test ./internal/fabricengine ./internal/pattern` and `lyx gate test --tags integration ./internal/fabricengine ./internal/battencli` passed before the self-check edits, which changed comments only.

## Gate repair

The gate flagged two fixed-column-wrapped comment lines this round added in `spawn.go` (the file header and the `PushWarpAt` doc).
Both were re-broken at sentence boundaries, one sentence per line.
A second gate run flagged one more line of the `PushWarpAt` doc, which split at a clause comma; it is now two sentences: "The detached push child's bypass handler drives DetachedPush instead." and "Nothing in shipped fabric ever runs gitrepo.PushCoalesced against a warp worktree."
