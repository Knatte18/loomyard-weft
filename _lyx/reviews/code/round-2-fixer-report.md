# Round 2 fixer-report

## Changes

- F1 (BLOCKING), fixed as proposed: `Run` in `internal/websterengine/runlevel.go` reads `AddedDecisionTexts` before `DeclaredPatternChanges`, so an unreadable record reaches the "read the parent decisions" refusal.
  Added the row "a decision record that cannot be read" to the refusal table of `TestRun`, with the record path set to a directory and the way-forward strings "transient" and "lyx webster run".
  Commit `4bbe8859`.
- F2 (MEDIUM), fixed as proposed: the seam assembly moved into `loomArbiterSeam` in `internal/loomcli/drivewiring.go`, which sets `JudgesFirst` for the Webster row only.
  `TestArbiterJudgesFirst` became `TestLoomArbiterSeam_JudgesFirst` and asserts through the built seam: `JudgesFirst` is non-nil, true for the Webster row, false for the others.
  `arbiterJudgesFirst` is removed, so the predicate is pinned once.
  Commit `973f71da`.
- F3 (LOW), fixed as proposed: the none row compares the master prompt rendered with nil decisions to the one rendered with `[]string{"none"}`, and the in-order row keeps only `first + "\n\n" + second`.
  Both stencil-heading anchors are gone.
  Commits `77711692` and `a5e4ab81` (the latter renames the in-order row to "master renders each parent decision in order", after the self-check).

`lyx gate test ./internal/websterengine ./internal/loomcli` was green after the three fixes.

## Self-check

- fix-consistency: no findings.
- fix-code-correctness: no findings.
- fix-affected-set: no findings.
- fix-proposal, LOW, the row name for F3 named a placement it no longer asserts: fixed as proposed (renamed, commit `a5e4ab81`).
- fix-proposal, LOW, no test pins the `goDriverDepsFor` call to `loomArbiterSeam`: disputed, `every fix worse`.

## Disputed

- Self-check finding of fix-proposal on F2, ground `every fix worse`.
  A test that reaches the call from `goDriverDepsFor` has to drive it, and it calls `os.Executable`, `tacklerFor` over the hub's config dirs and `arbiterDepsFor`.
  No test in `internal/loomcli` builds those today, and doing so makes a Tier-0 test depend on the host's lyx binary and hub configuration.
  The call is a single line, and the marking itself now lives in the tested builder.

## Deferred

None.
