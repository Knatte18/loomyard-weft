# Card 6 — calibration-engine-instruction-files

**Create:**
- `plan:tools/tokencount#websterInstructionFiles` -> `func websterInstructionFiles(reads ReadSet, run RunTally) []string`
- `plan:tools/tokencount#fakeInstructions` -> `type fakeInstructions []string`
- `plan:tools/tokencount#fakeInstructions.InstructionFiles` -> `func (f fakeInstructions) InstructionFiles() []string`

**Edit:**
- `tools/tokencount/main.go`
- `tools/tokencount#RunTally`
- `tools/tokencount#CountRun`
- `tools/tokencount#Calibrate` -> `func Calibrate(runs []RunTally, profile, configDir string, history PlanHistory, base BaseTrees, reads ReadSet) (Calibration, error)`
- `tools/tokencount#Calibration.addStarts` -> `func (c *Calibration) addStarts(run RunTally, history PlanHistory, base BaseTrees, instructionFiles []string) error`
- `tools/tokencount#Calibration.addFixed` -> `func (c *Calibration) addFixed(run RunTally, trees BaseTrees, instructionFiles []string) error`
- `tools/tokencount#readRunStartBase` -> `func readRunStartBase(run RunTally, history PlanHistory, trees BaseTrees, instructionFiles []string) (runStartBase, error)`
- `tools/tokencount#merriamTexts` -> `func merriamTexts(run RunTally, trees BaseTrees, instructionFiles []string) ([]string, string, error)`
- `tools/tokencount#withBaseFiles`
- `tools/tokencount#knownBaseContext`
- `tools/tokencount#TestCalibrate`
- `tools/tokencount#TestCalibrateStartFit`
- `tools/tokencount#TestCalibrateFailsOnAnUnusableProfile`
- `tools/tokencount#TestCalibrateGitBackedMatchesInMemory`
- `tools/tokencount#TestCountRun`

**Uses:**
- `tools/tokencount#fileAtBase`
- `plan:tools/tokencount#ReadSet`
- `plan:tools/tokencount#oneEngine`
- `internal/shuttleengine#RecordReads`
- `internal/shuttleengine#InstructionFileLister`

**Intent:** Calibration's Merriam texts take their instruction files from the Webster session's engine instead of `CLAUDE.md` by name (D8), so tokencount names no instruction file.

- `RunTally` gains `WebsterEngine string`, the name of the engine that located the run's first counted webster-role session in located order (engine, then session `ID`), empty when the run has none; `CountRun` sets it, so a run whose forks are counted carries it whether or not a Merriam start was measured.
- `websterInstructionFiles` returns the instruction file names of `run.WebsterEngine` through that engine's `Instructions` in the read set; an empty `WebsterEngine`, an engine absent from the set, and an engine whose `Instructions` is nil give none, so such an engine contributes no instruction texts.
- `Calibrate` takes the read set, computes the files once per run and passes them through `addStarts` and `readRunStartBase`, and `addFixed`, to `merriamTexts`.
- `merriamTexts` reads each named file at the run's base commit through `fileAtBase`, in the engine's order, in place of `CLAUDE.md`; an absent file adds nothing, as `CLAUDE.md` absent does today, so Claude's `CLAUDE.local.md`, untracked and absent at any base commit, adds nothing and an all-Claude run's calibration output is unchanged.
  Its doc comment names the engine's instruction files and keeps the untracked-file assumption, stated without a file name.
- `main.go`: `run` passes `reads` to `Calibrate`, and the package comment's base and fixed-context paragraphs name the Webster session engine's instruction files in place of `CLAUDE.md` and `CLAUDE.local.md`.

Tests over a read set of one engine `fake` whose `Instructions` is `fakeInstructions{"INSTRUCTIONS.md", "LOCAL.md"}`:
- `withBaseFiles` writes `INSTRUCTIONS.md` in place of `CLAUDE.md` and no `LOCAL.md`, and `knownBaseContext`'s comment names it; every based `RunTally` literal of `TestCalibrate` and `TestCalibrateStartFit` sets `WebsterEngine: "fake"`, which adds no Merriam start and so no fixed-context row.
- `TestCalibrate` keeps its figures, which pins that the engine's file is read at the base commit and the absent `LOCAL.md` adds nothing; it adds three rows whose base and fixed-context lines lack the instruction file's lines: a run with an empty `WebsterEngine`, one whose `WebsterEngine` is absent from the read set, and one whose engine has no `Instructions`; and a row whose engine names a file the base commit holds beside `INSTRUCTIONS.md`, whose lines add it.
- `TestCalibrateStartFit`, `TestCalibrateFailsOnAnUnusableProfile` and `TestCalibrateGitBackedMatchesInMemory` pass the read set; the integration test's repository commits `INSTRUCTIONS.md` in place of `CLAUDE.md`, and its in-memory run literal sets `WebsterEngine: "fake"`.
- `TestCountRun` asserts `WebsterEngine` on a run with webster sessions and its absence on one without.

**ImpactSummary:** the re-signed calibration members are package-private and their callers (`run` and the calibration tests) are on this card; `RunTally` gains a field that existing literals leave empty.

**Verify:** ! grep -rn 'CLAUDE' tools/tokencount

**Commit:** 6: calibration reads the Webster session engine's instruction files
