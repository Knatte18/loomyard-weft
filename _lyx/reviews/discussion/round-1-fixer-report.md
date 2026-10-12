# Fixer report — discussion, round 1

All edits are in `_lyx/discussion/decision-record.md`; `support-log.md` is unchanged.
Every finding's premise checked out against the code it cites (`internal/mergeresolve/mergeresolve.go:55`, `internal/fabricengine/merge.go:117`, `internal/landingshed/finalize.go:201-219`, `internal/landingshed/prgate.go:46`, both recipes' PR-Gate rows).

## Changes

- **F1** (fixed as proposed): D1 now says Merge-In fetches the parent's upstream, fast-forwards the local parent branch to it through `fabricengine`, merges the local branch, which now equals the upstream tip, and records that tip.
  A parent with no upstream uses its local tip.
  A local parent that is ahead or diverged is Stuck, with a way forward naming the push or reconcile in the parent's pair.
  Merge-In's own fast-forward answers who brings the local parent up after a hand push.
  D9 now reads the same ref D1 records.
  The hand-push acceptance criterion now requires that the rerouted Merge-In merges the pushed commits and that the next check passes.
  A new criterion covers an ahead or diverged local parent.
- **F2** (fixed as proposed): D5 adds a release on any lock-holding row's edge out of the window from Merge-In to Finalize, naming PR-Gate's rejection edge to PR-Rework (loom) and Darn-Write (darn), with Merge-In reacquiring.
  New acceptance criterion added.
- **F3** (fixed as proposed): D9 lets Finalize's merge-in-required retry run only while the parent's code side equals the recorded merged tip; otherwise it reroutes.
  New acceptance criterion added.
- **F4** (fixed as proposed): D5's exit hook is a compare-and-release on the loop's own pid and start time.
  D8's first bullet adds that batten releases its record once a resumed child stops again without having taken it back.
  The PR-Gate hand-over criterion covers that case.
- **F5** (fixed as proposed): a new Scope Out bullet says batten's Land row takes no parent lock, and why.
- **F6** (fixed as proposed): D6 and the acceptance criterion now name a review that ends without Merge-Bouncer's Done, such as an escalation followed by a goto past it.
- **F7** (fixed as proposed): D9 counts reroutes from the row's last Done or from the last resume of its reroute-cap halt, and that resume resets the count.
  D9's Bound and the fourth-reroute criterion now match.
- **F8** (fixed as proposed): the Scope Out bullet "A merge perch in darn." is deleted.

## Disputed

None.

## Deferred

None.
