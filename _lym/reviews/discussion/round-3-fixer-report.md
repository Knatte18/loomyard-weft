# Fixer report — discussion round 3

## Changes

- F1 (BLOCKING), fixed as proposed: applied the parent's two Added after Discussion entries.
  - D2: cut the staleness sentence and the Rationale clause that argued for it.
  - D5's Bound now says how long a dependency-drift miss lasts.
  - The D2 stale criterion now says a map stays fresh when only a linked file changed.
  - The Auto-mode assumption no longer includes the drift rebuild.
  - Open risks: the first question is gone, and the third now states the fresh-map dependency, the cadence and the raddle ordering.
  - Support log: reworded the rejected alternative, resolved Q10 and Q11, marked F4 of round 2 as answered, and added a Round 3 list.
- F2 (BLOCKING), fixed as proposed, then widened by the self-check:
  - D3's union holds only tests declared in test files the gate's host admits, read as `holdsTierTest` reads them.
    The rule is stated once for the whole union, so guard tests are covered too, not only the second member.
  - D6 puts the selected `-run` in the run's own args.
  - Added a D3 acceptance criterion for a test file constrained to another GOOS.
- F3 (MEDIUM), fixed as proposed: D2's two dirty checks are fresh worktree status reads.
  - D2 has a new Bound for an edit reverted between the two reads.
  - The hashes sentence now points at that Bound.
  - Added a D2 criterion. The self-check reworded it to "writes no new map and leaves any earlier one as it was".
- F4 (MEDIUM), fixed as proposed: the `inert_paths` exemption never covers a `go:embed` target of a linked package.
  Added a D5 criterion.
- F5 (LOW), fixed as proposed:
  - Added a D5 miss for an added top-level declaration whose name is a predeclared identifier.
  - Cut the dot-import clause of the Bound.
  - Qualified the Rationale and Q6.
  - Added `max` to the D5 miss examples.

## Self-check

- fix-consistency, MEDIUM: the host-admission filter did not cover the union's guard member. Fixed as proposed: the filter is stated once for the whole union.
- fix-consistency, LOW: the Turn 3 history lines on the drift rule and board-scope 1 had no pointer forward. Fixed as proposed: each now points to the round-3 ruling.
- fix-claims, LOW: the list of rebuild triggers in the dependency-drift Bound left out a testmap call naming the package. Fixed as proposed: added it.
- fix-claims, LOW: the guard member could still name another platform's tests. A duplicate of the fix-consistency MEDIUM, fixed there.
- fix-proposal, MEDIUM: the new D2 criterion said "without a map", but an earlier map stays on disk. Fixed as proposed: reworded.
- fix-way-forward, LOW: the same issue as the fix-proposal MEDIUM above; a duplicate, fixed there.
- fix-proposal, MEDIUM: cutting the staleness rule also cut "lacks the marker", so marker rollout waits for the age limit.
  Fixed as proposed: the Open risks bullet now says a map without the marker stays fresh, so it is rebuilt at the latest at the age limit, or at once by a testmap call that names the package.
  The parent's ruling removes "D2's staleness rule" as a whole, so I kept the cut and stated the bound instead.
- fix-proposal, LOW: the D3 rule names the tier's tag but not GOOS and GOARCH. The fork judged it no defect, since the reference to `holdsTierTest` settles it; no change.

## Disputed

None.

## Deferred

None.

## Rulings

None reached me this round. The parent's Added after Discussion entries were applied as F1.
