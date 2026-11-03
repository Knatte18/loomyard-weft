---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/run.go:138
    summary: the shared build running with the env module's variables, and the build's own env winning over them, has no test
    fix: In internal/gatecli, add a case to the existing gate test scenario in test_integration_test.go (the tier case added in this diff is the model) that configures a shared_build tier whose build step has its own env, writes an env.yaml with one name the build env also sets and one it does not, and asserts the build process sees the env module's name over an inherited value and the build's own value over the env module's. Reuse the stand-in record of the existing case; add no new test function.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/run.go:405
    summary: the hub branch of configDirs, which loads the hub file with this machine's override, is never run by a test
    fix: In internal/gatecli, add a case to the gate test scenario that runs inside a hub fixture built by hubforge, writes env.yaml in the board dir and a different value for the same name in the machine file (configengine.MachineFile), and asserts the tier sees the machine value. The existing case uses only the standalone worktree, so only the inHub == false branch runs today.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/driftcli/cli.go:204
    summary: no test fails if a Drift wiring site goes back to cfg.Step, so capture and compare receiving the variables is pinned only at the StepWithHubEnv unit
    fix: Add an env.yaml to an existing driftcli or loomcli drift wiring test and assert the step handed to the capture child (takeRun, runCaptureChild) and the step returned by driftDeps in internal/loomcli/driftwiring.go carries the env module's variables beneath the step's own env. Fold the assertion into the test that already drives each site; add no new test function per site. Also add a row to the existing StepWithHubEnv subtest in internal/driftengine/config_test.go for an env config that fails to load, asserting the error is returned and no step.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/reedengine/spawn.go:119
    summary: no test pins that launchStrandLocked puts the env exports ahead of the launch command, at every launch and relaunch
    fix: Extend the existing launchStrandLocked or stageLaunchScript test in internal/reedengine so a geometry with PaneEnv told leaves a staged launch script whose exports precede the launch command, on both dialects. Swapping the sh.Chain arguments at spawn.go:119 or dropping the line must fail it. TestPaneEnvExports pins only paneEnvExports.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/run.go:62
    summary: the gate refusing to run when the env config fails to load is not reached by any test
    fix: Add a row to the existing gate test scenario in internal/gatecli/test_integration_test.go that writes an env.yaml holding an invalid name such as BAD-NAME, runs lym gate test, and asserts a non-zero exit and an output naming the env config and its way forward.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/hubgeom/hubgeom.go:464
    summary: nothing asserts that the hub's reed geometry carries a PaneEnv returning the hub and machine values
    fix: Add an assertion to the existing hubgeom ReedGeometry test that the geometry built for a hub fixture has a non-nil PaneEnv and that calling it returns the hub file's values with the machine file's value winning per name.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/envconfig/config.go:283
    summary: a variable named PATH passes validation, and reed exports it after the prelude that prepends the spawning lym directory, so it replaces the prelude's PATH and breaks lym resolution in strand panes
    fix: In internal/envconfig/config.go LoadConfig, refuse PATH beside the LYM_ prefix with the same ErrInvalid message shape and a way forward, name it as a reserved constant next to reservedPrefix, state the refusal in the LoadConfig doc comment and in internal/envconfig/doc.go, and add a refused-name row for PATH to TestLoadConfig_StacksHubAndMachineName. Alternatively place the exports ahead of the prelude in composePaneLaunchLine's caller and keep PATH legal; pick one and record it in reedengine's doc.go.
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/envconfig/config.go:284
    summary: the LYM_ reservation compares case-sensitively, so on Windows a name such as lym_parent shadows lym's own variable
    fix: In internal/envconfig/config.go LoadConfig, compare strings.HasPrefix(strings.ToUpper(name), reservedPrefix) and add a refused-name row such as lym_parent to TestLoadConfig_StacksHubAndMachineName.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/gatecli/run.go:136
    summary: the env module's entries are appended after withBuildServerOff, so a hub variable can undo the build-server-off switches lym forces for every run, against the doc that the module replaces inherited values only
    fix: In internal/gatecli/run.go runTest, append hubEnv.Entries() to env before the withBuildServerOff call so the forced switches stay last, and say in internal/gatecli/doc.go that the module replaces inherited values and never the build-server switches. If the order is intended instead, state in the doc and in internal/envconfig/doc.go that the module may override the build-server switches.
---

### [MEDIUM] Shared build with the env module's variables is untested

**Location:** internal/gatecli/run.go:138
**Issue:** The decision record asks for the variables on the shared build and for a tier's or build's own env to win. The one gate test drives a tier through `renderTierEnv`. Removing the variables from the build path, or letting the module win over the build's `env`, fails no test.

### [MEDIUM] Hub branch of the gate's env config dirs is untested

**Location:** internal/gatecli/run.go:405
**Issue:** `configDirs` returns the hub's dirs with the machine override inside a hub. The one test uses a standalone worktree, so a regression that loaded only the anchor inside a hub, dropping the hub file and the machine override, passes every test. The machine layer is the point of the task.

### [MEDIUM] Drift wiring sites are not pinned

**Location:** internal/driftcli/cli.go:204
**Issue:** The three sites that hand a step to a run (`takeRun`, `runCaptureChild`, `driftDeps`) each call `StepWithHubEnv`. Only the unit is tested, so reverting one site to `cfg.Step` fails nothing. The load-failure error of `StepWithHubEnv` is also unpinned.

### [MEDIUM] Launch ordering of the pane exports is untested

**Location:** internal/reedengine/spawn.go:119
**Issue:** `TestPaneEnvExports` pins `paneEnvExports` alone. Dropping the `sh.Chain` line in `launchStrandLocked`, or swapping its arguments so the exports follow the launch command, fails no test.

### [LOW] Gate refusal on a broken env config is untested

**Location:** internal/gatecli/run.go:62
**Issue:** The refusal names a way forward but no test reaches it.

### [LOW] hubgeom PaneEnv closure is untested

**Location:** internal/hubgeom/hubgeom.go:464
**Issue:** Nothing asserts that a hub's reed geometry carries `PaneEnv` or that it returns the machine-over-hub values.

### [LOW] PATH is a legal variable name and replaces the prelude's PATH

**Location:** internal/envconfig/config.go:283
**Issue:** Reed exports the module's variables after its prelude, which prepends the spawning `lym` directory to `PATH`. `vars.PATH` replaces that entry, which breaks the property that a strand pane resolves `lym` to its spawning binary. Only `LYM_` names are refused. The value is operator-set and the machine layer is operator-owned, which keeps this LOW.

### [LOW] LYM_ reservation is case-sensitive

**Location:** internal/envconfig/config.go:284
**Issue:** Windows environment names are case-insensitive, so `lym_parent` shadows `LYM_PARENT` there while passing the reservation.

### [LOW] Env module entries follow the build-server switches

**Location:** internal/gatecli/run.go:136
**Issue:** `withBuildServerOff` removes inherited values of the switches and appends lym's own, and the module's entries follow. A variable such as `MSBUILDDISABLENODEREUSE` in `env.yaml` then wins, against the stated purpose that no build leaves MSBuild nodes behind.

## Rejected

- Lens code-affected-set finding on verify and card-verify paths getting no env-module variables (`verifytree/verify.go:388`, `cardverify/runner.go:63`, `battencli/land.go:182`): `verifytree.Verify` runs one `lym gate test --wait` call or such calls joined by `&&` (verify.go:300), and `cardverify` runs a line only when `IsGateCall` holds, so each test run goes through `runTest` and gets the variables; the design's enumeration names gate tiers, so these runs are covered by them.
- Lens code-tests NIT on the repeated `os.MkdirAll(dirs.Anchor, ...)` and the repeated final assertion in `TestSet_MachineLayerTakesAVariable`: a setup line and a closing read, not a second test covering the same behaviour, and the Set test pins the write the issue found refused.
