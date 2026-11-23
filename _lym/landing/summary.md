# Notify the parent of judged stops; keep reset and plan gate consistent

An amended plan could stall a run silently, and `webster reset` could leave the run tied to a code tip it had dropped.
This change closes three defects behind that stall.

## What changes

- **Stop notice.**
  A blocked stop that the tackler or the arbiter was asked to judge now sends the parent the "run stopped; stop report" line, as an unwatched stop already did.
  Before, a watched run blocked by Webster's auto-rebaseline sent the parent nothing.
  `internal/sheddriver` tracks whether the stop in hand was judged, keyed to that stop, so the flag cannot carry over to a later one.
  Batten still sends its own state-changed notice once the run parks, so the parent hears both at such a stop.
- **Plan gate.**
  `loomshed.ValidatePlan` raises a blocking `overview-frame-edited` finding once a batch has begun and `00-overview.md` outside its Card Index no longer hashes to the frame the run record holds.
  The gate now refuses the edit that Webster's rebaseline would refuse later.
  A record with no recorded frame, or with no begun batch, raises nothing.
  The finding is documented in `contracts/specs/loom-plan-checks-spec.md`, with the way forward: restore the overview and carry the change in a follow-up card after the last begun batch.
- **`webster reset`.**
  The records commit it makes now carries a `Run-Transition: <run id>` trailer next to its `Warp-SHA`, so the run's last transition names the new code tip.
  `lym loom resume` then no longer refuses on the dropped tip.
  Standalone mode has no run id and adds no trailer.
- **`fabricengine`.**
  New `Fabric.CommitWithTrailers` commits as `Commit` does and ends the weft commit's message with the given trailers.
  With a born warp and a trailer given, the commit lands empty when the files are unchanged.
  A trailer with a malformed key, or a blank or multi-line value, is refused before anything is staged.
  `webstercli`'s `fabricSync` passes trailers through to it.
- **Pattern doc.**
  `pattern/PATTERN-run-fsm.md` records that `webster reset` writes a transition and pushes the code first, then its records.

Closes #207

## Design decisions

- Only `webster reset` commits a transition naming the new tip.
  `lym loom goto` is unchanged: it writes only the status file and never moves the code tip, so it has no stale tip to correct.
  The issue asked for both.
- A plan with no recorded overview frame is left to the rebaseline rather than refused at the gate, because the rebaseline has a fallback to the baseline overview copy.
- The reset's code push goes first and its records commit follows, so until fabric's detached push lands the commit, origin's last transition names a tip origin no longer holds and the run waits.
  A failed push is owed and holds the run at its transition.

## To check by hand

Findings left open, to weigh before merging:

- `goto-transition-narrowing-unrecorded`: the narrowing of the issue's request to `webster reset` alone is recorded only in the decision above and is not noted where `lym loom goto` is documented.
- `judged-flag-outlives-its-stop`: the judged flag in `internal/sheddriver/drive.go` is keyed to the stop but cleared only at the next non-attempt step; check that no path hands back a stop under a stale flag.
- `no-card-index-only-edit-passing-row`: no test row shows an edit confined to the Card Index passing the new finding.
- `pattern-run-fsm-misses-reset-transition`: check that `pattern/PATTERN-run-fsm.md` names every place the reset's transition is written.
- `plan-checks-spec-lacks-overview-frame-edited`: check `contracts/specs/loom-plan-checks-spec.md` against the finding as `internal/loomshed/gates.go` raises it.
- `reset-unchanged-state-lands-no-transition` (blocking): when the reset leaves the records unchanged, check that its commit still lands, empty, carrying `Run-Transition`.
  That holds for a born warp; an unborn warp is not covered by the empty-commit rule.
- `watched-notice-premise-contradicts-batten-hold`: the comment on `driveNotices` says a watched batten sends its own notice, while a judged blocked stop now sends a second one; check that the doubled notice is intended.
- `wirehub-runid-untested`: `wireHub` in `internal/webstercli/wiring.go` sets the run id from `shedrun.ResolveRunID`, and no test pins that for a hub location.

## Merge danger

**Door:** two-way
**Blast radius:** webster
