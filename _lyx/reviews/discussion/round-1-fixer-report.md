# Fixer report — discussion round 1

## Changes

- F1, fixed as proposed: Open risks' first bullet now states that `reed-add-parentless-placement`'s card 8 sweeps only `SANDBOX-REED-SUITE.md` and `SANDBOX-REED-WATCH-SUITE.md`, so it edits none of the moved lines, and that its D4 removes the stacked strand layout C13 depends on, which leaves C13 needing a follow-up outside both tasks.
  Scope Out now says the parentless `lyx reed add` lines need no `--anchor` once #673's default changes, and makes no ownership claim.
  Verified against that task's `_lyx/plan/08-sweep-docs-and-sandbox.md` and its decision record's D4.
- F2, fixed as proposed: D1's controlled tmux exceptions bullet is replaced by one saying the creel suite has no such section, since no S9 or C scenario lists panes and C1's socket-directory listing is already a controlled creel exception.
  Scope In's first bullet now reads controlled creel exceptions.
  `support-log.md`'s Rejected alternatives gain the rejected tmux-exception carry-over.
- F3, fixed as proposed: D1's preamble bullet now has the how-to-run discovery line name `lyx creel` and `lyx reed` with their `--help`, and the capturing-findings JSON example's `ref` name a creel id such as `C2`.
- F4, fixed as proposed: D1's Pre-conditions add PowerShell 7, because reed's strands need it (the reed suite's own pre-condition 3 requires it) and every creel scenario adds reed strands.
- F5, fixed as proposed: the creel daemon risk now names its open lock and log file in the hub's creel state directory; `internal/creelengine/doc.go` puts the socket in the lyxtemp root.
- F6, fixed as proposed: the `shuttleSuite` teardown criterion is deleted, and Turn 2's entry now reads rejected, since `TestSuiteSpecs_ReedTeardownFlag` already pins it and no Decision changes `shuttleSuite`.

## Disputed

None.

## Deferred

None.
