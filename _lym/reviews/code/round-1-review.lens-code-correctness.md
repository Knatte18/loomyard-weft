```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced each function and branch the diff adds or changes with one concrete input.

**Per function or branch: the case I ran, and the result**

- `runRedFirst` (internal/gatecli/redfirst.go:99–160):
  - Empty `--gate`: refused with exit 1. Correct.
  - No hub: refused. Correct.
  - Scoper is `dotnet`: prints `note` and exits 0. **Not correct**, see finding F2.
  - `TierList` error: refused with exit 1. Correct.
  - Zero tiers: refused. Correct.
  - No parent branch: `verbDiffBase` refuses. Correct.
  - Diff touches only `calc.go`: `changedTestsAt` returns no tests, exit 4. Correct.
- `verbDiffBase` (mutate.go):
  - `--base` naming no commit: refused, labelled with the verb. Correct.
  - Merge-base read error: refused with "re-run". Correct.
- `redFirstCall.run` (redfirst.go:182–234):
  - `lymtemp` fails: exit 1. Correct.
  - SIGTERM during the run: returns 128+N and the deferred `RemoveAll` still runs. Correct.
  - Run exits 0: exit 3. Correct.
  - Run exits 1 with `TestOneIsTwo` named in the report: exit 0. Correct.
  - Run exits 1 with the report cut, or with a cause line: unjudged, exit 1. **Not correct**, see finding F1.
- `judgeRedFirst` (redfirst.go:262–279):
  - Exit 0: green. Correct.
  - Report naming the test: red. Correct.
  - Package failed outside any test: every changed test of that package is red. Correct.
  - Named red test beside a non-test report line (a cut, a timeout cause, unparsed output, or another package's build failure): `FailedTests` returns nil and the named red is lost. **Not correct**, see F1. The summary records only the cross-package build case as a bound.
- `changedTestsAt` (redfirst.go:284–316):
  - Changed `sub/a/a_test.go`: the package is read on both sides. Correct.
  - `testdata/x_test.go`: passed over. Correct.
  - A deleted `_test.go` alone: the package is read and no tests come back. Correct.
- `baseOverlay` (redfirst.go:324–344):
  - Changed `a.go` the base holds: maps to a copy of the base text. Correct.
  - `new.go` the base lacks: maps to `""`, which Go reads as deleted. Correct.
  - Rename `old.go→new.go`: `ChangedPaths` lists both paths, so `old.go` is restored and `new.go` deleted. Correct.
  - `_test.go`, a non-Go file, a file outside the module, a file under testdata: each left alone. Correct.
  - `baseFile` error: wrapped and returned. Correct.
- `writeOverlay`: the map with `""` values marshals under `Replace`. Correct.
- `testTargetsOf`:
  - Package at the module root: `.`. Correct.
  - Package `m/a`: `./a`. Correct.
- `testNamesOf` / `testLabels`:
  - The same name in two packages appears once in the `-run` names. Correct.
  - A same-named unchanged test in another target package also runs, and the judge filters it out by package. Correct.
- `PackagesFailedOutsideTests` (gateslot/runlog.go:414):
  - Header plus a `  pkg, failed outside any test:` line: returns `pkg`. Correct.
  - Four-space output line: skipped. Correct.
  - Cut line: stops. Correct.
  - Duplicate across tiers: kept once. Correct.
  - No report: nil. Correct.
- Green-record carry for the base run: `covered(…, run.args)` keys on args, and the args hold a fresh overlay path, so a HEAD-tree green record never skips the base run (green.go:343). Correct.
- `testreach.ChangedTests` / `isTestFunc` / `declaredFuncsWhere`:
  - Names `Test`, `Test_x`, `Test1`: tests. Correct.
  - Names `Testify`, `TestMain`, a method: not tests. Correct.
  - A test moved verbatim to another file: unchanged. Correct.
  - Parse error: an error naming the file. Correct.
  - `declaredFuncs` with `keep` always true: behaviour unchanged. Correct.
- `NewRedFirstGate` (loomshed/redfirstgate.go):
  - Labels `["feature"]`: passes. Correct.
  - Labels read error: returned error. Correct.
  - Record missing: returned error. Correct.
  - Added entry titled "no test seam: …": passes. Correct.
  - Verify reports dirty: `dirtyResult`. Correct.
  - Skipped on the same tree: passes. Correct.
  - Its `BaseCommand` is its own command. A pass prunes only entries for other trees, and the verify gate keeps its own entry the same way (verifygate.go:169). No inconsistent state.
- `redFirstFailure`:
  - Exit 3: non-terminal, green way forward. Correct.
  - Exit 4: non-terminal, No test seam way forward. Correct.
  - Exit 1: terminal. Correct.
  - Exit -1: terminal with the shell detail. Correct.
  - TimedOut: terminal. Correct.
- `entryLabels` (loomcli/wiring.go):
  - Board open error: wrapped. Correct.
  - Entry not found: nil labels, so the gate passes. Correct.
  - The board needs no close (boardengine/config.go:253). Correct.
- `resolvePlainGateClosure` `red-first`:
  - Relative `WorktreeRoot`, empty `VerifyDir`, relative `DecisionRecordPath`: each refused naming the field. Correct.
  - Nil seam: refused naming it. Correct.
  - Renders the call for the row's gate key. Correct.

**Findings**

- **F1 — MEDIUM, design.** internal/gatecli/redfirst.go:266 (`judgeRedFirst`).
  - Summary: a changed test that fails on the base by name is lost, and the run halts terminally, whenever the failure report holds any line that is not a named test.
  - Mechanism: `gateslot.FailedTests` returns `nil, false` as soon as it meets such a line, and `judgeRedFirst` discards `ok` and reads the nil list. Such lines include:
    - the cut line of a report past 80 lines;
    - a tier cause line such as a timeout of another tier;
    - a run with unparsed output;
    - another package's build failure.
  - Result: a correct red-first run is judged "none" (exit 1), and the loom gate fails it terminally.
  - Coverage: the documented bound names only the cross-package build case.
  - Evidence:
    - internal/gateslot/runlog.go:296–330: `FailedTests` returns `nil, false` on a non-test line, a non-run line or a cut line.
    - internal/gatecli/failurereport.go:30 and 205–215: the report is cut at 80 lines with a `... ` line.
    - internal/loomshed/redfirstgate.go:137–139: the default branch is terminal.
- **F2 — MEDIUM, design.** internal/gatecli/redfirst.go:120 (`runRedFirst` scoper branch).
  - Summary: in a hub whose scoper is not `go`, the verb exits 0, the code that means a test went red.
  - Result: `NewRedFirstGate` records it as `StatusPassed` and passes every `bug` run of such a hub with no test judged and no "No test seam" record. That is the silent pass the board entry forbids.
  - Evidence:
    - redfirst.go:119–121 returns `output.Ok` with exit 0.
    - internal/loomshed/redfirstgate.go:104–105: Passed and Skipped pass.
    - The board entry's Goal: "never a silent pass".

**Tool calls**

1. `internal/gatecli/green.go:325–395` and the `BaseCommand` grep: to see whether a green record could skip the base run, and how the verify gate sets `BaseCommand`. My context had neither.
2. `internal/gateslot/runlog.go:286–330` and the vet/guard grep: to see `FailedTests`' contract when `ok` is false, and whether the explicit-target run plans vet or guard steps. My context lacked both.
3. `internal/loomshed/verifygate.go:120–172` and the report-cap grep: to compare the verify gate's `BaseCommand`, and to find where the failure report is cut. My context lacked both.
4. `internal/gatecli/failurereport.go:290–326`: to see how the cut line is rendered. My context lacked it.
5. `internal/gatecli/failurereport.go:24–36` and `180–215`: to find the report's line cap and its cut rule. My context lacked them.
6. `boardengine.OpenHub` and a peer caller: to check whether an opened board must be closed. My context lacked it.
