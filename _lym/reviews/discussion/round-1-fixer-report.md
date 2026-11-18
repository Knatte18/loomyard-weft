---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
---

# Round 1 fixer-report — discussion

F5 is `decision`-class and held by the parent; no ruling reached this round.

## Changes

- F1: D3 takes option (a).
  The overview reads `reed.json` under the worktree's anchor `.lym` directory through `reedengine.LoadState` and finds the `loom-status` strand.
  It names the window by the strand's `Display.Window` group when `WindowGroups` records that group as shared, else by the role `loom-status`.
  The derivation is pinned to `createOwnWindowPaneLocked` and `planOwnWindowPlacement`.
  The Rationale drops the claim about avoiding recomputing reed's rule and explains the `window_share_max` 0 case through the shared record.
  The D3 criterion now names the strand's window name, and a new criterion pins the shared group and the not-shared, `window_share_max` 0 case.
- F2: D1's re-add goes through `AddStrandUnless` with the loom status role as its unless-name, which orch's strand seam gains.
  It is pinned to `AddStrandUnless` and `strandNameLocked`, and a skip counts as done and writes no sidecar.
  The Constraints bullet names the skip instead of a role refusal.
  The D1 criterion reads: when a strand holding the role exists, `ensureRunsStrand` adds none and returns no warning.
- F3: D1 states that the loom bootstrap's add in `ensureStatusStrand` goes through `AddStrandUnless` on the same name, and that a skip counts as kept.
  A Scope In bullet names that bootstrap change, and a D1 criterion covers the race between the bootstrap's read and its add.
- F4: D1 states that the re-add runs once the replace succeeded, whatever the runs sidecar write returns, and that the two warnings are joined.
  A failed `ReplaceStrand` on this branch names `lym loom start` as well as `lym beam start`.
  The re-add failure now leaves only the runs strand in place.
  Two D1 criteria cover these paths.
- F6: The Constraint now says the overview takes reed's shared read lock on `reed.json.lock` for its one reed state read, through `ReadJSON` in `internal/state/state.go`, and no other lock.
- F7: D1 cites its evidence: every `orchcli.BeamUp` caller passes the beam's own location, and `wireCLI` sets the orch's location and reed geometry from it.
  In the support log, the existing-paths 2 line under Turn 3 now credits `wireCLI`.
- F8: The D1 failure criterion covers the re-add or its status sidecar write failing.
- F9: The structural clause is cut from the first D2 criterion.
- Support log: the interactions 1 line under Turn 3 notes the `AddStrandUnless` change, and a Round 1 entry under Review rounds lists the dispositions.

## Self-check

- fix-consistency: no findings.
- fix-claims: no findings.
- fix-way-forward: no findings.
- fix-proposal, MEDIUM: cutting F9's clause left no criterion that checks the spec the bootstrap produces. Fixed as proposed with a new D2 criterion: with a runs strand present, `lym loom start` adds the status strand with the own-window anchor, `KeepViewer` and the recipe's window group.

## Disputed

None.

## Deferred

None.

## Rulings

None.
