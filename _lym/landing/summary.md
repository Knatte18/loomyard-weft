# Prove a bug fix's test was red first, and name its root cause

A darn run of a board entry labelled `bug` now holds the writer until a test its diff adds or changes fails on the merge base.
A fix whose tests all pass without the fix proves nothing about the bug, and the gate refuses it.
The fix commit also carries a `Root-Cause:` trailer, which Code-Review checks the way it checks `Loosened-Check:`.

## What changes

- `lym gate red-first --gate <key> [--base <rev>]` is a new verb in `internal/gatecli`.
  It finds the tests the diff adds or changes through the new `testreach.ChangedTests`.
  It runs them once per package on the base's code: the base's Go files go in as an overlay, and the tests keep their head text.
  It exits with a code from `internal/gateslot` that names the verdict:
  - green, when every test passed on the base;
  - no test, when the diff adds or changes none;
  - unjudged, when the hub's scoper is not `go` or the diff changes a non-Go file the overlay cannot put back.
- `NewRedFirstGate` in `internal/loomshed` is a must-pass gate that runs the verb through `verifytree.Verify`.
  The new `red-first` gate sits on the `Darn-Write` row of `contracts/recipes/darn-recipe.yaml`, after `verify` and before `drift`.
  `loomcli/wiring.go` wires its two readers: the entry's labels and the rendered command.
  An entry without the `bug` label passes at once and runs nothing.
- A bug with no seam for a test is a stop the record names.
  The parent records it with `lym loom decision add` under a title opening `No test seam`, and the gate then passes.
  The writer's failure text names that way forward for the no-test and unjudged verdicts.
- A run that timed out, crashed its runner or exited with another code judged none of the diff's tests, and fails the gate terminally.
- The writer stencil `darn-template-write.md` gains a section on bug entries, and its `Root-Cause:` trailer is named there.
  `loom-rubric-code-review.md` makes a trailer that names a symptom, or a defect the diff does not touch, a finding.
  It makes a missing trailer a finding only when the decision record alone is the answer key, which only the darn recipe uses.
- `gateslot` holds the exit codes, the `bug` label and the `No test seam` title as shared constants.
  The mutate verb's base resolution is now `verbDiffBase`, shared with the new verb.
  `docs/overview.md`, `pattern/PATTERN-gate-slot.md` and the package docs describe the gate.

## Design decisions

- The gate runs in Darn-Write only.
  Tapestry and sampler runs of a bug entry are not checked: tapestry gates per Webster batch, a different seam with its own design questions.
- The check runs each package's tests under that package's tier and base overlay, so a test of one package never runs by another's name.
- A Go-only judgement: on another scoper the verb says so and exits with the unjudged code, so the stop goes to the parent instead of passing silently.
- The gate has no fixer, since only the writer can make a test show the bug.

## To check by hand

No integration or end-to-end run of a real bug entry through Darn-Write was made.
Check these findings, which a review left open:

- `internal/gatecli/redfirst.go`: the base run keeps non-Go inputs, such as testdata a changed test reads, at their head text.
- The `bug` label and the `No test seam` title are repeated as literals outside `gateslot`.
- On a hub whose scoper is not `go`, check that `lym gate red-first` exits with the unjudged code and not zero.
- `judgeRedFirst`: in a mixed report, a failure the output names may be dropped from the verdict.
- No behaviour test covers the entry-labels reader beyond `internal/loomcli/entrylabels_integration_test.go`.
- The gate covers darn runs only, and nothing but this description records that narrowing.
- The rubric's missing-trailer clause may reach past its intended scope of a decision-record-only answer key.
- The verify-site tripwire and the docs may not name the new gate's site.
- The Merge danger blast radius below may be too narrow.

## Merge danger

**Door:** two-way
**Blast radius:** darn
