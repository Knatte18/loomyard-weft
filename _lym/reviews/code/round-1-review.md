---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/driftengine/seats.go:136
    origin: lens:code-correctness
    summary: "captureReading still tells the Drift-Baseline agent the capture runs in its dir under the anchored directory; with dir no longer defaulted to '.', an unset dir renders as empty backticks and a set dir is described against the removed base"
    fix: "In internal/driftengine/seats.go captureReading, stop printing step.Dir raw under 'under the anchored directory'. Either pass the worktree root and anchor into captureReading (through BaselineInputs, which BaselineTable already receives) and print the directory gateslot.ResolveCommandDir resolves, shown relative to the worktree root, or word it as 'run in <dir> relative to the worktree root, or in the anchored directory when no dir is set' and branch on an empty step.Dir so no empty backticks render. Add a BaselineTable test row with an empty Dir and one with a relative Dir that asserts the prompt text."
  - id: F2
    severity: MEDIUM
    class: consistency
    location: internal/battencli/lifecycle_integration_test.go:1568
    origin: lens:code-affected-set
    summary: "stepPairDirNamesTheTaskWorktree still asserts InnerRun.PairDir equals AnchorPath and says 'own anchor', but the seam now returns WorktreePath; the test passes only on a root-anchored hub and cannot fail if the seam reverts"
    fix: "In internal/battencli/lifecycle_integration_test.go rename stepPairDirNamesTheTaskWorktree's comment to say the task worktree root, assert got equals taskLocation.WorktreePath(), and run the step over a hub fixture anchored at a subdirectory so WorktreePath and AnchorPath differ; if hubforge has no such fixture, add one row to a notice test that renders a path under a subdirectory anchor and asserts it keeps the leading subdirectory."
  - id: F3
    severity: MEDIUM
    class: scope
    location: internal/configcli/hubwide.go:450
    origin: lens:code-tests
    summary: "copyStagedToHub now runs checkHubModule with CheckDirs on a hub-wide editor save, but no test saves a drift or gate file with a missing dir or shows a save allowed when the file already failed the same dir check"
    fix: "In internal/configcli/configcli_test.go add two TestDispatchHubWideBoard editor-save rows next to the darn rows: a drift editor save whose staged file sets capture.dir to a directory missing in the prime is refused with the hub file unchanged and the staging copy kept, and a save over a hub file whose dir already failed the same check, staged with an unrelated change, is written."
  - id: F4
    severity: MEDIUM
    class: scope
    location: internal/driftengine/compare.go:80
    origin: lens:code-tests
    summary: "No test pins that RunCompare, RunSideCapture, or the driftcli and loomcli request wiring pass the worktree root as WorktreeDir; dropping or swapping it in any of them fails no test"
    fix: "In internal/driftengine/compare_test.go add a TestRunCompare row with the anchor at <root>/dotnet and Step.Dir 'dotnet' that runs 'pwd -P' and asserts it ran in <root>/dotnet. In internal/driftengine/capture_test.go add the same row for RunSideCapture. For the wiring, add one assertion in a driftcli or loomcli test that a captured CaptureRequest or CompareRequest carries WorktreeDir equal to the worktree root and AnchorDir equal to the anchor on a subdirectory-anchored fixture."
  - id: F5
    severity: MEDIUM
    class: scope
    location: internal/driftengine/config.go:239
    origin: lens:code-tests
    summary: "driftengine TestCheckDirs declares only a capture step, so the compare entry of CheckDirs is unpinned and could be dropped with every test green"
    fix: "In internal/driftengine/config_test.go TestCheckDirs add a row seeding compare as {command: 'c {baseline_dir}', dir: gone, timeout_min: 1} that wants an error naming drift config key 'compare.dir' and the resolved path; the row's seed must set compare through withKey like the capture rows do."
  - id: F6
    severity: LOW
    class: design
    location: internal/driftengine/steprun.go:486
    origin: lens:code-correctness
    summary: "RunSideCapture refuses a missing dir with the baseline capture's way forward (lym drift capture and lym loom resume) because both pass Name capture, though a side capture is a different verb"
    fix: "In internal/driftengine/steprun.go missingDirRefusal, branch the 'again' text on the site as well as the name: for the side capture site name only the side-capture verb (the one internal/driftcli/cli.go runSideCapture serves), not lym loom resume. Add a row asserting that text to the side-capture test."
  - id: F7
    severity: LOW
    class: consistency
    location: internal/driftcli/doc.go:14
    origin: lens:code-affected-set
    summary: "driftcli doc still says the engine is told the anchored directory where a command step's relative dir is joined on; the base is now the worktree root"
    fix: "In internal/driftcli/doc.go line 14 say the engine is told the worktree root, which a command step's relative dir is joined onto, and the anchored directory, where the capture child starts and a step with no dir runs."
  - id: F8
    severity: LOW
    class: scope
    location: internal/gateslot/config.go:544
    origin: lens:code-tests
    summary: "StatCommandDir refuses a dir that exists as a file through info.IsDir, but no row covers a dir resolving to a regular file"
    fix: "In internal/gateslot/config_test.go TestCheckDirs add a row whose tier dir names a regular file created under root, wanting the refusal that names the key and the resolved path."
  - id: F9
    severity: LOW
    class: consistency
    location: internal/gateslot/config.go:526
    origin: handler
    summary: "ResolveCommandDir is documented as the one rule every hub command resolves its dir by, but the gate's own run path still resolves through WithDefaultDir and gatecli resolveRunDir, two more implementations of the rule that nothing ties together"
    fix: "In internal/gateslot/config.go reword the ResolveCommandDir comment and the matching sentence in internal/gateslot/doc.go so each says the gate run path reaches the same directory through Config.WithDefaultDir and its run-dir join, and ResolveCommandDir is the rule drift steps and the hub-wide dir checks use; or have gatecli's resolveRunDir path and WithDefaultDir call ResolveCommandDir. Do not leave a claim of one rule with no link."
  - id: F10
    severity: LOW
    class: decision
    location: internal/gateslot/config.go:550
    origin: handler
    summary: "The Goal asks that the config loader refuse a missing dir; the diff reads this as the CheckDirs check on hub-wide writes only and leaves LoadConfig accepting one, a reading the decision record does not carry"
    fix: "Leave the code in place and defer this finding to the run's parent: recording the reading, or refusing it, is lym loom decision add by the parent and lies outside the fixer's write surface."
---

### [MEDIUM] Baseline agent's prompt still names the old base

**Location:** internal/driftengine/seats.go:136
**Issue:** `captureReading` renders the declared capture as run in the step's `Dir` under the anchored directory. This diff made `Dir` relative to the worktree root and stopped defaulting it to `.`, so the Drift-Baseline agent now reads the wrong base for a set dir and empty backticks for an unset one. The agent judges the suite's reach from that sentence. No test asserts it.

### [MEDIUM] PairDir test asserts the old seam and cannot catch a revert

**Location:** internal/battencli/lifecycle_integration_test.go:1568
**Issue:** The seam moved to `WorktreePath()` in `wire.go` and `dobbynotice.go`. The step still asserts `AnchorPath()` and its comment still says the task worktree's own anchor. It passes only because the fixture hub is anchored at the root, where the paths are equal, so the notice change the Goal asks for (keep the full resolved path) has no row that fails if it reverts.

### [MEDIUM] Hub-wide editor save of a missing dir has no test

**Location:** internal/configcli/hubwide.go:450
**Issue:** `copyStagedToHub` now checks command dirs through `checkHubModule`. The new rows cover `--set` only. Dropping `CheckDirs` from the editor path, or breaking the already-failing-same-error allowance for a dir, fails nothing.

### [MEDIUM] WorktreeDir wiring in compare, side capture and the CLIs is unpinned

**Location:** internal/driftengine/compare.go:80
**Issue:** Only `RunCapture` has a row showing a relative dir joined onto the worktree root under a subdirectory anchor. `RunCompare`, `RunSideCapture` (whose test uses an empty dir, which resolves to the anchor whatever `WorktreeDir` holds) and the five request constructions in `driftcli` and `loomcli` have none, so the Goal's one base is unpinned for compare and for every wiring path.

### [MEDIUM] CheckDirs compare entry is unpinned

**Location:** internal/driftengine/config.go:239
**Issue:** Every `TestCheckDirs` row declares `capture` only. The `compare` entry of the checked steps could be removed with every test green.

### [LOW] Side capture is refused with the baseline capture's way forward

**Location:** internal/driftengine/steprun.go:486
**Issue:** `RunSideCapture` passes `Name: stepCapture`, so a missing dir there ends in `lym drift capture and lym loom resume`, the recovery of the baseline capture, not of an operator's side capture.

### [LOW] driftcli doc keeps the old base

**Location:** internal/driftcli/doc.go:14
**Issue:** The doc says the anchored directory is where a step's relative dir is joined on, which this diff made untrue.

### [LOW] File-not-directory case of StatCommandDir has no row

**Location:** internal/gateslot/config.go:544
**Issue:** The `info.IsDir()` half of the check is pinned by no test.

### [LOW] Claim of one rule over three implementations

**Location:** internal/gateslot/config.go:526
**Issue:** The comment and `gateslot/doc.go` call `ResolveCommandDir` the one rule every hub command resolves its dir by. The gate's run path still computes its directory through `WithDefaultDir` and `resolveRunDir` (`internal/gatecli/tierrun.go:403`), which is equivalent today but is a separate implementation no test or comment ties to it. The summary's statement that the gate's run path is left alone is borne out; the doc's claim of one rule is not.

### [LOW] Loader refusal read as a hub-wide write check

**Location:** internal/gateslot/config.go:550
**Issue:** The Goal's Expected bullet says `lym config <module> --set` and the config loader refuse a missing dir. The diff and its summary read "loader" as the `CheckDirs` check on hub-wide writes and editor saves, for reasons the summary gives (`LoadConfig` runs in every worktree). The reading is sound, but the decision record does not carry it, so it is an unrecorded choice for the parent.

## Rejected

- Focus item on the notice change doing more than the Goal licenses: the commit message and summary state that every notice path now renders against the worktree root; `pairRelative` keeps a doubled `dotnet/dotnet` whole, and the Goal's second Drift bullet asks for exactly that. Not a defect.
- Focus item on a missing `dir` key for a module other than drift and gate: no other module declares a command `dir` in its template, so none is left out.
- Focus item on the carry-over account for existing hub configs: the summary's account matches the Goal's fourth bullet (it says how `dir: .` carries over and why no migration runs).
- Focus item on `checkHubModule` loading the config twice: `CheckLoad` runs first and `CheckDirs` only on success, so a failing file is not loaded twice and the extra load on a passing file is not a defect.
