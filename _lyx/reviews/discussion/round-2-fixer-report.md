# Fixer report — discussion, round 2

## Changes

- F1 (BLOCKING), fixed as proposed: D4's two session-end orders now put the refusal probe ahead of `End`, and D4 adds "A removal the refusal probe refuses ends no owned process."
  I checked this against `Teardown.EndSession`, which runs `t.refusal` after the quiet wait and before the loop end (`internal/pairteardown/teardown.go`).
  A new acceptance criterion covers it: a refused `fabric remove` of a paused run, such as a dirty pair without `--force`, ends no owned process and leaves the session live.
- F2 (BLOCKING), fixed as proposed: D7 now exempts a Finalize past its parent-side merge, read as `keepsLockForFinalize` reads it, at every owned-elsewhere site.
  - Such a run keeps its parent lock and finishes its landing, and the pair's removal ends its processes.
  - D7's Bound is extended to match.
  - A new Fact states that `ownedElsewhere` keeps the parent lock for such a Finalize and that a parent-lock holder passes a slug pause entry (`SlugEntryExempt`).
  - The owned-elsewhere acceptance criterion now says that run is not ended at the probe, the wake or `lyx orch start`.
- F3 (MEDIUM), fixed as proposed: `reed down`'s not-running reading in D4 now adds "no live driver strand as the quiet rule reads it".
  The `reed down` acceptance criterion now says a run with a live loom driver strand and no pause is refused with nothing ended.
- F4 (MEDIUM), fixed as proposed: D4 now states that today `reed down` in a pair kills a running run's session, and that the new refusal withdraws that.
  D9's Bound now names that `reed down` in a pair no longer reaches the session end for a run that reads running and unpaused.

## Self-check

- fix-consistency, LOW: Q16 in the support log's question ledger still said "every owned-elsewhere pause" and did not name D7's new Finalize exception.
  Fixed as proposed: Q16 now names the exception.
- fix-claims, LOW: D7's parenthetical left out that a failed read of either seam also keeps the lock (`internal/battenshed/innerrun_owner.go` `keepsLockForFinalize` doc).
  Fixed as proposed: the parenthetical now ends with "or a failed read of either seam".
- fix-claims, LOW: the wake's `holdRuns` and `lyx orch start`'s `pauseOwnedRuns` have none of the reads `keepsLockForFinalize` uses.
  Fixed with my own change: the claim was true, so I left the Decision text alone.
  I added a note for the plan writer that those two sites need a per-slug status read and a parent-work read.
- fix-proposal, LOW: D9's new Bound, "until a pause", is narrower than D4's readings.
  Fixed as proposed: it now reads "until one of D4's readings holds, such as a pause".
- fix-way-forward: no findings.

## Disputed

None.

## Deferred

None.
