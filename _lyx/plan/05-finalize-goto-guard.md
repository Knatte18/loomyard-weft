# Card 5 — finalize-goto-guard

**Create:**
- `plan:internal/loomrecipe#finalizeEngine` -> `const finalizeEngine = "Finalize"`
- `plan:internal/loomrecipe#IsFinalizeRow` -> `func IsFinalizeRow(row string) (bool, error)`
- `plan:internal/loomrecipe#TestIsFinalizeRow` -> `func TestIsFinalizeRow(t *testing.T)`
- `plan:internal/loomcli#finalizeDirtGuard` -> `func finalizeDirtGuard(isFinalize func(row string) (bool, error), parentDirt func() (checkout string, paths []string, mergeRecord bool, err error)) shedengine.GotoGuard`
- `plan:internal/loomcli#composeGotoGuards` -> `func composeGotoGuards(guards ...shedengine.GotoGuard) shedengine.GotoGuard`
- `plan:internal/loomcli#loomCLI.parentCodeDirtFromFabric` -> `func (c *loomCLI) parentCodeDirtFromFabric() (checkout string, paths []string, mergeRecord bool, err error)`
- `plan:internal/loomcli#loomCLI.finalizeGuard` -> `func (c *loomCLI) finalizeGuard() shedengine.GotoGuard`
- `plan:internal/loomcli#TestFinalizeDirtGuard` -> `func TestFinalizeDirtGuard(t *testing.T)`

**Edit:**
- `internal/loomcli#loomCLI.settledGuard`
- `internal/loomcli#loomCLI`
- `internal/loomcli#newLoomCLI`
- `internal/loomcli/doc.go`

**Uses:**
- `internal/loomrecipe#BouncerRunSubdir`
- `internal/loomcli#settledSegmentGuard`
- `internal/loomcli#parentBranchSeam`
- `internal/loomengine#LoomReviewsDir`
- `plan:internal/landingshed#DirtyParentRefusal`
- `plan:internal/fabricengine#Fabric.CodeDirtyPaths`
- `internal/fabricengine#Fabric.MergeInProgress`
- `internal/fabricengine#OpenParent`

**Intent:**
D4: a goto onto Finalize is refused while the parent's code checkout has tracked dirt, before any driver steps it.

- `IsFinalizeRow` reports whether the named row of the loom or darn recipe has engine `Finalize`, scanning both recipes as `BouncerRunSubdir` does; an unknown name is false, and a recipe parse failure is a `loomrecipe:`-prefixed error.
  The row is identified by its engine, never its name.
- `finalizeDirtGuard` is the Finalize arm, a pure function over its two reads.
  A target `isFinalize` rejects is admitted with no parent read; an `isFinalize` error is returned, which Shed turns into a refusal naming the retry.
  For the Finalize row it calls `parentDirt`: an error is logged through `logger.Warn` and admitted, since Finalize's own steps then report the real problem;
  a fabric merge record on the parent pair is admitted whatever the list holds, since that dirt may be another run's merge in flight, and Finalize's own guard then words it with no path list (card 3);
  an empty list is admitted; otherwise it refuses with `landingshed.DirtyParentRefusal(checkout, paths)`.
  Every decision is made here, so the read below only reads.
- `loomCLI.parentCodeDirtFromFabric` reads through `c.recordedParentBranch(c.location)` and `fabricengine.OpenParent`, then both `MergeInProgress` and `CodeDirtyPaths`, returning all three answers: the parent's code checkout alone, through `fabricengine`, never `Fabric.DirtyPaths`.
  Any read error is returned.
- `loomCLI` gains a `parentCodeDirt func() (checkout string, paths []string, mergeRecord bool, err error)` seam field, which `newLoomCLI` sets to `c.parentCodeDirtFromFabric`, as it sets `recordedParentBranch`; a test substitutes canned answers, since an untagged test cannot build a real parent pair.
- `loomCLI.finalizeGuard` is `finalizeDirtGuard(loomrecipe.IsFinalizeRow, c.parentCodeDirt)`.
- `composeGotoGuards` runs the guards in order and returns the first refusal or error, admitting when all admit.
- `loomCLI.settledGuard` returns `composeGotoGuards(c.finalizeGuard(), settledSegmentGuard(…))`, so every goto it already guards passes the Finalize arm: an orch `lyx loom goto`, the tackler's and the arbiter's verdict check and their consume.
  Its godoc says it is the run's whole goto guard, the Finalize arm first, then the settled-segment arm.

Bound: the arm acts only on the Finalize row and only reads the parent's code checkout; it moves no run and writes no file (`shedengine.GotoGuard`'s contract), and a goto onto any other row reaches the settled-segment arm unchanged.
Its refusal is the refusal spec's transient class: the dirt clears by a named action, and the text names the goto and the resume as the retry.
It decides on the dirt present at goto time; Finalize's own merge guard stays the authority.
A tackler resume naming Finalize under a dirty parent is rejected with this text by the guard check `TacklerCall.resumePrecondition` already runs, and its consume refuses likewise, so no step runs and no second halt note is written; `TestTacklerCall_VerdictOutcomes`' refusing-guard row and `TestGoDriver_Settle`'s consume-refusal row already pin that path for any refusal text.
`internal/loomcli/doc.go` describes the goto guard's two arms where it names the told goto guard.

Tests:
- `TestIsFinalizeRow`: `Finalize` is true; `Merge-In`, `PR-Gate` and `No-Such-Row` are false, all with a nil error.
- `TestFinalizeDirtGuard`, table-driven over `composeGotoGuards(finalizeDirtGuard(loomrecipe.IsFinalizeRow, fakeDirt), fakeSettled)` with a counting `parentDirt` fake and a `fakeSettled` that refuses only `Plan-Burler-Review` with `the Plan-Review verdict is settled` and errors only on `Webster-Burler-Review`:
  `Finalize` with dirt `"/hub/parent"`, `a.go`, `b c.go` refuses with text holding `/hub/parent`, `"a.go", "b c.go"`, `lyx loom goto --to Finalize` and `lyx loom resume`;
  `Finalize` with an empty list is admitted;
  `Finalize` with a merge record beside a non-empty list is admitted;
  `Finalize` with a read error beside a non-empty list is admitted;
  `Plan-Burler-Review` returns the settled refusal unchanged and reads no parent;
  `Webster-Burler-Review` returns the later guard's error;
  `Merge-In` is admitted and reads no parent;
  and an `isFinalize` fake returning an error makes the guard return it.
- The same test's wiring rows run over a hand-populated `loomCLI` whose location is a temp hub, whose `routing.Producers` holds the Plan-Review segment, whose reviews directory (`loomengine.LoomReviewsDir` joined with the `Plan-Bouncer` run subdirectory `BouncerRunSubdir` names) holds a settled record, and whose `parentCodeDirt` seam returns the dirt above:
  `c.settledGuard()` refuses `Finalize` with the dirt text and `Plan-Burler-Review` with the settled text, so the run's goto guard composes both arms;
  `c.finalizeGuard()` refuses `Finalize` with the dirt text and admits `Plan-Burler-Review`, so the arm card 6 wires into the driver is the Finalize arm alone.
- Two rows reach `parentCodeDirtFromFabric` with no git: with `recordedParentBranch` set to `parentBranchSeam(err)` it returns that error, and with `parentBranchSeam(nil)` over a location whose anchor is no repository `OpenParent` fails and it returns that error; in both, `c.finalizeGuard()` with the production seam admits `Finalize`.

**ImpactSummary:** `settledGuard` keeps its signature and its four callers (step spec, arbiter deps, arbiter consume, driver deps) now get the composed guard; `loomCLI` gains one seam field set in `newLoomCLI`; only a goto onto the Finalize row changes, and only by a parent read.
