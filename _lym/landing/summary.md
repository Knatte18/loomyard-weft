# Red-first gate: a "No bug fixed" record lifts a tests-only entry

Closes #184

A `bug`-labelled entry that only adds golden tests pins existing behaviour and fixes no bug.
Its tests pass on the merge base by design, so the red-first gate's "fails on base, passes with fix" check could never be met.
The writer had no valid way forward, spent its attempts on the same finding and escalated to a human.

## What changes

- The parent can now record an added decision under a title opening `No bug fixed`, with `lym loom decision add`.
  The red-first gate then passes at once and runs nothing, the same way it already does for a title opening `No test seam`.
  The new title is `gateslot.NoBugFixedTitle`; `loomshed` compares it case-insensitively with the existing one.
- The gate's green-exit finding no longer tells the writer to invent a bug.
  It names both ways forward: make a test fail on the base when the change fixes a bug,
  or ask the parent for the `No bug fixed` record when it fixes none.
  The first sentence now reads "none of them fails before the change" instead of "none of them shows the bug the fix fixes".
- `lym gate red-first` states the same second path in its own green-exit error.
- The darn writer stencil tells the writer to ask its parent for the record before writing code when the entry fixes no bug.
- The Discussion template, its review rubric and the discussion spec let a bug entry's record state that it fixes no bug
  in place of naming a failing test or a missing seam.
- The darn recipe comment, `docs/overview.md` and the `gatecli`, `loomshed` and `shedrecipe` package docs name the new title.

## Design decisions

- The exemption is a parent-recorded decision, not an inference from the diff.
  Only the parent's added entry lifts the gate; a Sketch's own statement that the entry fixes no bug does not,
  matching how `No test seam` already works.
- The `No test seam` ask is unchanged in wording except that both asks now come from one helper, `parentRecordAsk`, taking the title.
- `redFirstCall` gains a `runTiers` field so a test can replace the real tier runs.
  `runRedFirst` sets it to `runTest`, so production behaviour is unchanged.

## Tests

- `TestRedFirstGate_VerdictPerLabelRecordAndExitCode` has a row for a `No bug fixed` record passing without running the check,
  and its green-exit row pins the new finding.
- `TestRedFirstCallRun_JudgesTheTierRunsOfTheChangedTestsOnTheBase` drives `redFirstCall.run` through the injectable runner
  for a red test, a green test and a run that judged none.
- The `TestGateRedFirst` integration case checks the green-exit message names the `No bug fixed` title.

Nothing was left to check or run by hand.

## Merge danger

**Door:** two-way
**Blast radius:** loomshed
