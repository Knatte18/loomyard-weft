# Round 4 fixer report — discussion

## Changes

- F1 (fixed as proposed): D7 names `lyx loom start`, run in the task worktree, as the restart at the owner-probe site, and states that `lyx loom resume` refuses a running child with no driver, since its Go spawn is for a halted run only, with a refusal naming `lyx loom start`.
  Added a Constraints Fact citing `internal/loomcli/resume.go` `noLiveDriverMessage` and `bootstrap.go` `mustSpawnDriver`.
- F2 (fixed as proposed): D7's restart is split by site.
  The owner-probe site keeps the surviving batten's driver-dead notice, then `lyx loom start`.
  At the hibernate wake and `lyx orch start` sites, `End` ends batten too, so nothing restarts the chain on its own until `lyx adopt` step 5 (`battencli.ResumeInWindow`) starts a batten that spawns the unconfirmed running child again, with no second driver; a run left owned elsewhere gets no restart.
  The acceptance criterion now pins the adopt respawn for the wake and `lyx orch start` sites and the notice plus `lyx loom start` for the probe site.
  The respawn evidence is recorded as a Constraints Fact (`internal/battenshed/doc.go`, `internal/adoptcli/doc.go`).
- F3 (fixed with the proposal's first option): D6 states that the gate reads the parent lock record at each network call through `HoldsParentLock`, not at spawn, and that without more a push reaching its call after the record is released would put back its flags and exit.
  It then decides that the spawner reads the record at spawn and flags the child; for its whole life the flagged child never exits for coverage and passes its own slug's owner entry at the gate.
  D6's Bound limits the flag to the pushes a Finalize spawns past its parent-side merge.
  The acceptance criterion pins a push that reaches its call after the record is released.
  Added a Constraints Fact on batten's record release (`internal/battenshed/innerrun_parentlock.go`), a support-log rejected alternative for deferring the push, and ledger entry Q17.
  The deferred option was rejected: under an owner pause on a pair another machine owns, nothing on this machine carries the landing's commits.

## Self-check

- fix-proposal, MEDIUM: the whole-life flag let a flagged push pass any slug entry, an operator's slug pause set after the release included. Fixed differently from the fork's wording: the flag now passes only a slug entry carrying `pauserecord.OwnerCaller` of the push's own slug; any other slug entry is judged by the live record, as today.
- fix-consistency, LOW: substituting the flag for `HoldsParentLock` would make the in-flight marker's `UnderParentLock` and `lyx tender status` report a lock no record holds. Fixed by the same change: `HoldsParentLock` stays the live reading, and D6 says the marker reports the live record.
- fix-claims, LOW: "batten releases the record once Finalize is Done" holds only for a record handed to batten. Fixed as proposed: "the record is released or lapses once Finalize is Done".
- fix-way-forward, LOW: "nothing restarts the chain until `lyx adopt`" overstated, since `lyx batten run <slug>` and `lyx loom start` restart it by hand once the run reads owned here. Fixed: "on its own", with the manual restarts named.

## Disputed

None.

## Deferred

None.
