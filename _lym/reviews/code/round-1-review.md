---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: lens:code-affected-set
    location: internal/webstercli/acceptaudit.go:108
    summary: Nine production files the diff changes are not gofmt-clean, because each new fabricSync call has no space after the comma before its label argument, and the repository's gofmt guard will fail at Parent-Align.
    fix: "Run `lym loom fmt <file>` on internal/webstercli/acceptaudit.go, beginbatch.go, fixplan.go, fixverify.go, rebaseline.go, recordbatch.go, recoverbatch.go, restoreplan.go and run.go, so every `c.syncOptions,\"...\"` or `c.syncOptions,fmt.Sprintf(...)` reads `c.syncOptions, ...`. Then confirm that `gofmt -l internal/webstercli internal/loomcli` prints nothing."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/webstercli/wiring_test.go
    summary: No test pins that wireHub and wireStandalone read FABRIC_SKIP_GIT, FABRIC_SKIP_PUSH and the strand-name variable into syncOptions and strandName, so deleting either assignment passes every test.
    fix: "In an existing serial wireHub case of internal/webstercli/wiring_test.go and an existing serial wireStandalone case, call t.Setenv for FABRIC_SKIP_GIT=1, FABRIC_SKIP_PUSH=1 and agentname.StrandNameEnv=<a literal name> before wiring. After wiring, assert c.syncOptions.SkipGit and c.syncOptions.SkipPush are true and c.strandName equals the literal. Add these as assertions or rows of the existing tests, not as a new test function."
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/landing/summary.md:50
    summary: The change description says loomcli's after runs stay at or below the better before run, but its own table shows after run 2 at 8.2 s against the better before run at 7.4 s.
    fix: "In _lym/landing/summary.md, replace the sentence `Its after runs stay at or below the better before run.` with one the table bears out. For example, state that after run 1 (5.4 s) is below both before runs and after run 2 (8.2 s) lies between them, so the loomcli gain is within the load swing."
  - id: F4
    severity: MEDIUM
    class: scope
    origin: lens:code-scope
    location: _lym/landing/summary.md:22
    summary: The entry asks webstercli to pass in the environment and working-directory values its tests set and then mark those tests parallel, but the tests that set XDG_STATE_HOME, LOCALAPPDATA or LYM_TRACE, or call t.Chdir, stay serial, and no Added after Discussion entry records the narrowing.
    fix: "Leave the code in place and defer this finding to the run's parent, who rules whether the narrowing stands. The record takes no new entry once this review has begun, so if the parent wants those tests made parallel, the parent files that work as a board note for a new task. Until then the summary's `Still serial` bullet stays as the writer's account."
  - id: F5
    severity: LOW
    class: scope
    origin: lens:code-scope
    location: internal/loomcli/gatefixer_integration_test.go:18
    summary: The entry asks loomcli to share one built hub copy between the tests that only read it, yet every moved test that only reads its hub, such as TestPairSlugOf, still takes its own CopyHub instead of hubforge.SharedHub.
    fix: "In internal/loomcli, switch each moved test that never writes its hub from hubforge.CopyHub(t, pairHubShape) to hubforge.SharedHub(t, pairHubShape), starting with TestPairSlugOf in gatefixer_integration_test.go. Keep CopyHub in every test that commits, writes files or upserts board entries."
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/hubfixture_helper_test.go:30
    summary: The diff removed the space between creates and the path in the seedLoomConfig doc comment, so it now reads creates<anchorPath>/_lym/config/loom.yaml.
    fix: "In internal/loomcli/hubfixture_helper_test.go, restore the line to `// seedLoomConfig creates <anchorPath>/_lym/config/loom.yaml with the embedded template's contents.`"
  - id: F7
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/webstercli/verbs_test.go:11
    summary: The rewritten file header says that every test whose verb reaches a fabricSync call builds its fixture with newSkipGitVerbsFixture, but the failing-opener way-forward tests reach fabricSync through newVerbsFixture with zero syncOptions.
    fix: "In the internal/webstercli/verbs_test.go file header, say that a test whose verb reaches a working fabricSync builds its fixture with newSkipGitVerbsFixture, and that the way-forward tests reach it through a failing opener instead."
  - id: F8
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/ownercheck_integration_test.go:54
    summary: TestRunOwnerVerdict spells hubforge.Shape{Anchor ".", Pairs [slug]} inline, while every other moved loomcli test uses the package's pairHubShape.
    fix: "In internal/loomcli/ownercheck_integration_test.go, replace hubforge.CopyHub(t, hubforge.Shape{Anchor: \".\", Pairs: []string{slug}}) with hubforge.CopyHub(t, pairHubShape)."
---

### [BLOCKING] Nine production files left unformatted

**Location:** `internal/webstercli/acceptaudit.go:108`, and the changed `fabricSync` calls in `beginbatch.go`, `fixplan.go`, `fixverify.go`, `rebaseline.go`, `recordbatch.go`, `recoverbatch.go`, `restoreplan.go` and `run.go`.

**Issue:** Each rewritten call reads `fabricSync(c.openFabric, c.anchorRel, c.syncOptions,"…")`, with no space after the comma.
`gofmt -l internal/webstercli internal/loomcli` lists all nine files, and the base `acceptaudit.go` is clean.
`TestGofmt_ModuleFilesAreFormatted` (`cmd/lym/gofmt_test.go:49-51`) carries `//lym:only parent-align finalize`.
The darn-write gate that ran on this tree therefore never ran it, and Parent-Align will fail on these files.
`reset.go:186` is formatted correctly, which shows the other call sites were edited by hand.

### [MEDIUM] The wiring-time read of the environment is untested

**Location:** `internal/webstercli/wiring.go:218-220` and `:357-359`; the tests belong in `internal/webstercli/wiring_test.go`.

**Issue:** Before the change, the verbs read `FABRIC_SKIP_GIT`, `FABRIC_SKIP_PUSH` and `LYM_STRAND_NAME` at call time.
Tests that set those variables therefore exercised the production read.
Every such test now sets `syncOptions` or `strandName` on the `websterCLI` it builds.
No test reaches `c.syncOptions = fabricengine.EnvSyncOptions()` or `c.strandName = os.Getenv(agentname.StrandNameEnv)`.
Deleting either line in either wiring path passes the suite.
Production would then ignore the skip switches, and the orphaned-Master guard and Master's pause claim would treat every caller as an unnamed shell.

### [MEDIUM] A loomcli measurement claim the table contradicts

**Location:** `_lym/landing/summary.md:50`

**Issue:** The table lists the loomcli before runs at 17.8 s and 7.4 s and the after runs at 5.4 s and 8.2 s.
The sentence "Its after runs stay at or below the better before run" is false for after run 2: 8.2 s is above 7.4 s.

### [MEDIUM] Part of the webstercli ask is left undone without a recorded narrowing

**Location:** `_lym/landing/summary.md:22`; the tests are in `internal/webstercli/wiring_test.go`, `cli_integration_test.go`, `cli_tmux_test.go` and `cli_test.go:41-44`.

**Issue:** The entry says webstercli's tests run serially because they set the environment or working directory, and asks to "pass those values in instead, then mark the tests parallel."
The diff does this for the skip switches, the strand name and `PATH`.
The tests that set `XDG_STATE_HOME`, `LOCALAPPDATA` or `LYM_TRACE`, or call `t.Chdir`, stay serial.
The change description gives the reason: "production reads those through code this change leaves alone."
That narrows the amended entry, and no `Added after Discussion` entry records it.
Ruling on it is the parent's call, so the finding is deferred to the parent rather than fixed by the fixer.

### [LOW] Tests that only read their hub still copy it

**Location:** `internal/loomcli/gatefixer_integration_test.go:18`, and other moved tests that never write their hub.

**Issue:** The loomcli ask is to "share one built hub copy between the tests that only read it."
The diff shares the built pair template, but every test still takes its own `CopyHub`.
That includes tests that only resolve locations and read from the hub, such as `TestPairSlugOf`.
`hubforge.SharedHub` exists to hand such a test the template itself (`internal/hubforge/template.go:5`).

### [LOW] Damaged seedLoomConfig doc comment

**Location:** `internal/loomcli/hubfixture_helper_test.go:30`

**Issue:** The hunk that added `pairHubShape` also changed the unrelated `seedLoomConfig` comment from "creates <anchorPath>" to "creates<anchorPath>".
No decision asks for that edit, and it leaves the comment broken.

### [NIT] verbs_test.go header overstates which tests use the skip-git fixture

**Location:** `internal/webstercli/verbs_test.go:11`

**Issue:** The header says that every test whose verb reaches a `fabricSync` call builds its fixture with `newSkipGitVerbsFixture`.
`TestBeginBatchCmd_FabricSyncFailureWayForward`, `TestRecordBatchCmd_FabricSyncFailureWayForward` and `TestRecoverBatchCmd_FabricSyncAndReedBootWayForward` also reach it, but through `newVerbsFixture` and a failing opener.

### [NIT] ownercheck spells the pair shape inline

**Location:** `internal/loomcli/ownercheck_integration_test.go:54`

**Issue:** This is the only moved loomcli test that does not use `pairHubShape` (`hubfixture_helper_test.go:371`), although it copies the same shape.

## Rejected

- The code-tests lens's side note that the folded `TestWireLanding_FillsTheLandingSeamsPerRecipe` no longer checks language skills under the empty default recipe is not a defect. Both rows still assert the language skills and the PR-Review seams, so no assertion is dropped or loosened.
- The code-correctness lens's open question about `TestRunOwnerVerdict` is rejected. It asked whether the pair template could first be built after `boardkit.InstallReplay` registers the Bolt replayer. Before the change the same `AddPair` already ran after that registration, on the copy, and no evidence shows that the replay changes `AddPair`'s result.
- The pre-existing gap that no `TestResetCmd` row would fail if `ResetPairCode` got zero options was not introduced by this diff. As the code-tests lens noted, the remote update ignores `SkipGit`.
