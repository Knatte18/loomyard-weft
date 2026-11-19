# A red-first gate and a Root-Cause trailer for bug runs

A darn run whose board entry is labelled `bug` can no longer hand off a fix whose test would have passed before the fix.
Darn-Write gains a `red-first` gate entry, after `verify`, that runs the tests the diff adds or changes once on the merge base's code and fails when every one of them passes there.
The writer's prompt asks for a `Root-Cause:` trailer on the fix's commit, and the Code-Review rubric checks it against the diff.
The design is candidate 2 of `docs/research/survey/pocock-skills-2026-11-19/report.md`.

## What changes

- **`lym gate red-first --gate <key> [--base <rev>]`** (`internal/gatecli/redfirst.go`).
  It finds the tests the worktree's diff adds or changes with `testreach.ChangedTests`,
  builds a Go `-overlay` that puts the merge base's version of every changed production `.go` file of the module back (a file the base lacks is removed),
  and runs the key's tiers in process over those tests' packages with `-run` matching exactly their names.
  It exits 0 when one of them fails on the base, by name or by its package failing to build there;
  `gateslot.RedFirstGreenExit` (3) when every one passed; `gateslot.RedFirstNoTestExit` (4) when the diff holds no test;
  and 1 when the run judged none of them, such as after a timeout.
  A hub whose scoper is not `go` judges nothing and exits 0 with a note.
  It runs only the key's tiers: none of the key's lints, scope or report steps.
- **`testreach.ChangedTests`** returns the top-level test functions of the `_test.go` files that are new or whose declaration text differs,
  matched across the two sides by package and name as `ChangedFuncs` matches functions.
  The file walk is shared with `ChangedFuncs` through `declaredFuncsWhere`.
- **`gateslot`** holds what the verb and its site share: `RedFirstCall`, the two exit codes,
  and `PackagesFailedOutsideTests`, the read of a failure report's packages that failed outside any test.
- **`loomshed.NewRedFirstGate`** runs the call through `verifytree.Verify`, so a pass on a tree is recorded and skipped on the next arrival.
  An entry without the `bug` label passes at once and runs nothing.
  So does a decision record holding an added entry whose title opens `No test seam`: the parent's recorded stop for a bug no test can reach.
  Exit 3 and exit 4 fail with the writer's way forward; any other failure judged no test and fails terminally, so the run halts for the parent.
- **`shedrecipe`** adds `red-first` to the gate vocabulary over two new Env seams:
  `EntryLabels`, which loom wiring fills with a read of the run's board entry at each arrival,
  and `RedFirstCommand`, filled with `gateslot.RedFirstCall`.
  The darn recipe lists the gate on Darn-Write between `verify` and `drift`, with 3 attempts and no fixer.
- **Stencils.** The darn write prompt gains a "A bug entry" section: write the test so it fails on the merge base,
  ask the parent for a `No test seam` record when no test can reach the bug, and put `Root-Cause: <defect>` on the fix's commit.
  The Code-Review rubric gains a "Root cause of a bug fix" bullet beside the Loosened checks one.
- **Docs.** The package docs of `gatecli`, `gateslot`, `testreach`, `loomshed` and `shedrecipe`, and `docs/overview.md`, describe the verb and the gate.

## Design decisions

- The base run uses a Go overlay over the working tree rather than a second worktree at the merge base.
  The new tests exist only at HEAD, so the run needs HEAD's test files over the base's code, which an overlay gives without touching git or the fabric boundary.
- The gate reads the verb's exit code rather than parsing its output, so the contract between the two is the pair of `gateslot` constants.
- A changed test that does not build on the base counts as red.
  The record's rule is that the gate fails when every test passes there, and a test that does not build does not pass.
- "A bug with no seam for a test is a stop the record names" is met by an added decision titled `No test seam`.
  The writer cannot run `lym loom decision add`, so its way forward is to ask the parent, which keeps the stop in the record rather than in the writer's hands.
- Only Darn-Write carries the gate.
  The record speaks of a gate step for a `bug` entry; tapestry and sampler runs of such an entry are not checked.

## Bounds

- `go.mod`, `go.sum`, embedded files and other non-Go files are not put back to the base.
- A changed test that every tier of the key skips counts as passing on the base.
- A tier that prebuilds `lym` builds it from the working tree, so a test that runs the binary runs HEAD's.
- A run whose failure report mixes a changed test's failure with another package's build failure is read as judging nothing, which halts the run.

## Merge danger

**Door:** two-way
**Blast radius:** darn
