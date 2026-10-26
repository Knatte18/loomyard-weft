# Fixer-report, code review round 1

## Changes

- **F1 (BLOCKING), fixed as proposed, then narrowed.**
  `PATTERN.md` `PATTERN-shed-stop-chain` now says a blocked stop at a row the driver's arbiter seam marks as judged first goes rule table, arbiter, then parent.
  The review's "(Webster's)" parenthetical was dropped in a follow-up commit after the fix-proposal lens, because it pins one consumer of a generic seam.
- **F2 (MEDIUM), fixed with my own change.**
  A `TestRun_MasterSpawn` row sets `Geometry.DecisionRecordPath` to a record holding an `Added after Discussion` entry and asserts the master prompt carries it.
  Not made: the directory-path row that asserts the new unreadable-record refusal.
  `DeclaredPatternChanges` reads the same path first (`internal/discussionparser/patternchanges.go:311-316`) and returns every non-not-exist read error with its own re-entry way forward, so a directory path stops there and never reaches the new `AddedDecisionTexts` branch in `Run`.
  That branch is reachable only through a race between the two reads, so no test can drive it.
- **F3 (LOW), fixed as proposed.**
  The `master renders none` row asserts `"\n\nnone\n\n## A plan-drift"` and `notContains` the unfilled marker, with no stencil prose.
  The review's `inOrder` variant failed: `inOrder` matches the first occurrence of `"\nnone\n"`, which sits earlier in the stencil.
- **F4 (LOW), fixed with my own change.**
  The review's proposal needs a test that builds `goDriverDepsFor`, and none exists; `arbiterDepsFor` needs a real config and registry no loomcli test sets up.
  The marking is extracted to `arbiterJudgesFirst` in `internal/loomcli/drivewiring.go`, and `TestArbiterJudgesFirst` pins it: Webster true, other rows and the empty name false.
  The assignment line that wires it into the seam is still not pinned.

Commits: one per finding, plus one for the PATTERN narrowing.
`lyx gate test` is green for `./internal/loomcli`, `./internal/websterengine` and `./cmd/lyx`.

## Disputed

None.

## Deferred

None.

## Self-check

- fix-code-correctness, LOW: the unreadable-record row for F2 is not covered. Fixed with my own change: reason given under F2.
- fix-proposal, LOW: the wiring line for F4 is not pinned. Fixed with my own change: reason given under F4, no test builds the deps.
- fix-proposal, LOW: the PATTERN entry names Webster's row. Fixed as proposed: the parenthetical was dropped.
- fix-proposal, note outside its lens: the `Run` read-error refusal has no test. Same evidence as F2, nothing further to do.
- fix-consistency: no findings.
- fix-affected-set: no findings.
