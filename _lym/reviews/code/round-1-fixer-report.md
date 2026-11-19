---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed-differently
    reason: The proposed rule refuses every changed non-Go path inside the module's packages, and the module root here is the repository root, so a change to docs/overview.md would exit unjudged and send the parent a No test seam ask for a bug that has a seam. baseOverlay refuses a changed non-Go path only when a Go file lies in its directory or above it up to the module root, since only such a package can embed or read it. go.mod and go.sum stay a stated bound.
  - id: F5
    disposition: ruling
    reason: The parent's ruling accepts the narrowing to darn. The code already carries the gate on Darn-Write alone and the change description records that, so nothing changed and no flag went to the parent.
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed-differently
    reason: The proposed wording, "in a darn run", names a recipe the shared rubric never shows its reader (its header says tapestry, darn and sampler read it alike). The clause now turns on the answer key's form, a decision record alone, which only the darn recipe uses (darn-recipe.yaml:45; tapestry-recipe.yaml:437 names the record and the plan).
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
---

# Round 1 fixer-report

## Changes

Commits, in order. Findings that share a file share a commit, since a file's edits cannot be split by finding without interactive staging.

- `verifytree`, F6: `planVerifySites` and its `lym:watch` line in `cmd/lym/verifiedtree_test.go` list `internal/loomshed/redfirstgate.go`.
  `PATTERN-gate-slot`'s Verified tree section names the Darn-Write red-first gate as a site and its call, and `Verify`'s doc names the `lym gate red-first` call.
- `loom-rubric-code-review`, F7: the missing-trailer clause applies when the answer key is a decision record alone.
- `loomcli`, F9: `entrylabels_integration_test.go` builds a hub pair and asserts `entryLabels` returns no labels with no entry, then each relabelled entry's labels at each call.
- `shedrecipe`, F12: `TestResolveGateSpec_RedFirstRendersTheCallOfTheRowsOwnGateKey` asserts the key `darn-write` reaches `env.RedFirstCommand`.
- `gatecli`, F1 F2 F3 F4 F10 F11 F13:
  - F1: `gateslot.NoTestSeamTitle` and `gateslot.BugLabel` are the one declaration; `loomshed` and `gatecli` reference them, and the no-test refusal formats the title with `%q`.
  - F2: `gateslot.RedFirstUnjudgedExit` (5) is the exit of a hub whose scoper is not go. `redFirstFailure` has a non-terminal case with the parent ask.
    The verb's Long help, the three package docs, `docs/overview.md` and the summary match.
    Rows: `TestGateRedFirst_AHubWhoseScoperIsNotGoExitsUnjudged` and a `TestRedFirstGate_VerdictPerLabelRecordAndExitCode` row for exit 5.
  - F3: `gateslot.NamedFailedTests` reads every named failure up to the cut line; `judgeRedFirst` uses it. `TestNamedFailedTests` and two `judgeRedFirst` rows cover it.
  - F4: `baseOverlay` returns the changed paths it cannot put back, and `run` exits `RedFirstUnjudgedExit` before any tier. Rows in `TestBaseOverlay_*` and an integration subtest cover it.
  - F10: `TestChangedTestsAt_*` gains `sub/c`, whose only changed path is its test file.
  - F11: rows for the lower-case title, the unreadable record and the tier timeout line.
  - F13: `run` runs the tiers once per package of the changed tests and judges each package's own output. `TestTestsByPackage_*` and an integration subtest with a same-named failing test in a second package cover it.
- Self-check follow-ups: `darn-template-write.md` and `shedrecipe/doc.go` name the unjudged check; `judgeRedFirst` ignores a name under a run that is no tier or guard run; the `red` field comment no longer promises changed-test order.
- F8: `_lym/landing/summary.md` names the Code-Review perch of every loom recipe in the blast radius, and states the new exit, the per-package runs and the changed bounds. It lives under `_lym`, so it carries no commit.

- Gate repair: the comment lines the gate listed for length or a wrapped break are rewritten one sentence per line, under 120 characters.
  The files are `gatecli/doc.go`, `gatecli/redfirst.go`, `gateslot/doc.go`, `gateslot/redfirst.go`, `gateslot/runlog.go`,
  `gateslot/failedtests_test.go`, `loomcli/entrylabels_integration_test.go`, `loomshed/redfirstgate.go` and `shedrecipe/doc.go`.
  No code changed.
- Reach repair: `TestNoSeamWayForward_NamesTheRecordTheParentAddsUnderTheGatesTitle` covers `noSeamWayForward`, which no mapped test reached.
  The other unreached functions in the gate's note, `runMutate` and `verbDiffBase`, are not this round's code, and the verb's own paths run in the tier-1 tests the map does not hold.

## Self-check

- fix-code-correctness, MEDIUM, no test drives the per-package loop of `run`: fixed with an integration subtest holding a failing same-named test in a second target package.
- fix-code-correctness, LOW, `PATTERN-gate-slot` says `lym gate red-first` takes its own slots: disputed, premise false. `internal/gatecli/run.go:3` says `runTest` takes each tier's gate slots in turn.
- fix-code-correctness, LOW, the F4 refusal is broader than an embed or read: disputed, every fix worse. A narrower rule needs `go:embed` parsing, and a miss ships the false green F4 forbids; `internal/gatecli/redfirst.go` `holdsGoFileAtOrAbove` states the rule and its row pins it.
- fix-consistency, LOW, the darn writer stencil lacks the unjudged case: fixed.
- fix-consistency, LOW, `shedrecipe/doc.go` lacks the unjudged hold: fixed.
- fix-proposal, MEDIUM, the rubric scoping names a recipe the rubric cannot see: fixed, see F7.
- fix-proposal, LOW, `judgeRedFirst` counts a name under a non-tier run: fixed with a `Tier >= 0` condition and a row.
- fix-proposal, LOW, the `red` field comment promises changed-test order: fixed.
- fix-proposal, LOW, the non-terminal scoper refusal costs attempts until the parent records the decision: disputed, every fix worse. A terminal result would stop the run with no ask the parent can answer; `internal/loomshed/redfirstgate.go` `redFirstFailure` gives the ask.
- fix-affected-set: no findings.

## Disputed

None of the review's findings.

## Deferred

None.

## Rulings

- F5: the parent's ruling (decision record entry "Red-first gate on darn runs only") accepts that the gate runs in Darn-Write only.
  The target already holds it, so no change was made; the ruling is neither mistaken nor in need of a flag, and the parent was not told.
