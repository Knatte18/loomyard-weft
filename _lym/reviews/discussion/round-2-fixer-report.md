---
dispositions:
  - id: F19
    disposition: fixed-differently
    reason: "The held path is added as proposed, except for the ledger write. The proposal writes a ledger landing only when none of the run stands and the parent's upstream lacks the commit. Instead the held path makes its remaining records' `Deps.LandLocally` call with its own landing commit, which writes the ledger landing when none stands, as a re-entered Finalize's `pushParent` does today (internal/landingshed/finalize.go:1047-1054). That call also commits the run's status records and marks its pair's push (internal/loomcli/landingdeps.go:926-943, 959), so skipping it would leave the run's final records unpushed."
  - id: F20
    disposition: fixed-differently
    reason: "The proposal does not call `Deps.LandLocally` while a ledger landing of the run stands. Instead the call is kept and given the run's own landing commit. Skipping it would also skip the run's status-record commit and its pair's push mark (internal/loomcli/landingdeps.go:937, 959). A re-recorded landing keeps the earlier records commit while origin lacks it (`oldestHeldRecords`, internal/loomcli/landingdeps.go:978-1011), and `runpusher.RecordLanding` rewrites it by slug with the same tip (internal/runpusher/parent.go:719-725). So the ledger landing written at the move keeps its tip and records hold. The rest is as proposed: `Deps.RecordLanding` and the empty-diff check take the run's own commit, the Constraints line covers the leader's own run, and the criterion is added. The criterion is split between `internal/landingshed` and `internal/loomcli` after the self-check."
---

## Changes

All changes are in `_lym/discussion/decision-record.md`.

- **F19 (fixed differently).**
  D8's opening paragraph now sends a run whose work the parent holds, whatever its entry says, to a held path.
  The held path never merges the parent into the task worktree and runs no gate.
  It judges the owner checks as a held run does today, by the reconcile's ended and stopped verdicts alone, then runs `handedOverStop`.
  It does its remaining landing records with its own landing commit (D6), removes its entry and ends Finalize.
  It names how the run's own landing commit is found, with a fallback to the parent's head that records no landed size.
  The `landed` bullet now takes the held path.
  The D8 criterion now reads: a run whose work the parent holds, whatever its entry says, runs no merge and no gate, writes no landing commit and ends Finalize.
  The ledger write differs from the proposal: see the disposition's reason.
- **F20 (fixed differently).**
  D6 now says a batched run's remaining landing records take its own landing commit, never the parent's head, for `Deps.RecordLanding`, the landed size, the empty-landing check and `Deps.LandLocally`.
  It says how a re-record keeps the ledger tip and the records commit.
  The Constraints' `pushledger.Landing` line names the leader's own run and the later `LandLocally` call.
  Two acceptance criteria were added:
  - seam arguments, Tier 1, `internal/landingshed`;
  - the ledger and the ahead-by-owed read, Tier 1, `internal/loomcli`.

## Self-check

- fix-consistency, LOW: the held path's parent-head fallback took a landed size from another run's commit. Fixed as proposed: the fallback records no landed size and reads the landing as not empty, as a held run with no merged parent head does today.
- fix-way-forward, MEDIUM: the new D6 criterion asserted the ledger contents and the ahead-by-owed read in `internal/landingshed`, which reaches them only through seams. Fixed as proposed: the criterion is split into a `landingshed` seam-argument criterion and an `internal/loomcli` ledger criterion.
- fix-proposal, MEDIUM: the held path took an entry's `landed` commit even when the parent's head no longer holds it. Fixed as proposed: that commit is taken only while the parent's head holds it.
- fix-proposal, MEDIUM: the trailer lookup named no range and no rule for which commit wins, and today's reader sees only commits past upstream (internal/fabricengine/landing.go:111-131). Fixed as proposed: the lookup takes the newest matching commit on the parent's first-parent history, origin's included.
- fix-claims, LOW: the `oldestHeldRecords` claim omitted its descent condition. Fixed as proposed: the sentence names both conditions.

## Disputed

None.

## Deferred

None.

## Rulings

None.
