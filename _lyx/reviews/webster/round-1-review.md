---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/gatecli/cli_test.go:47
    summary: "TestGoTestArgs_AssemblesDirCapTagsPackagesAndFlagsInOrder tests the unexported goTestArgs helper, and TestGateTest_Scenario already asserts the exact go argument list through the CLI"
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/websterengine/cardgate_test.go:189
    summary: "new tests TestRenderBatchGate and TestRerunCardVerifies_Slotted touch no process-global state yet skip t.Parallel, and TestPublishVerify_FailureRecord skips it with no comment naming the global state it touches"
  - id: F3
    severity: BLOCKING
    class: scope
    location: internal/gatecli/run.go:57
    summary: "the gate refusal rows for an unreadable or invalid gate.yaml and for a go binary that cannot start have no test reaching them"
  - id: F4
    severity: LOW
    class: consistency
    location: internal/gateslot/config.go:30
    summary: "LoadConfig detects an absent gate.yaml by matching error text, which configengine says not to do, and the result stacks two different reconcile commands as the way forward"
  - id: F5
    severity: LOW
    class: design
    location: internal/loomshed/publishfailure.go:61
    summary: "checkedPublishFailure keeps a log path whose file name is free text and a test package that may start with a dash, so record text reaches the review prompt and the go command line unchecked"
  - id: F6
    severity: LOW
    class: consistency
    location: pattern/PATTERN-config-strictness.md:6
    summary: "the strict-caller list omits internal/gateslot although cmd/lyx/configstrictness_test.go now pins it as a strict caller"
  - id: F7
    severity: LOW
    class: scope
    location: internal/configcli/configcli.go:507
    summary: "runMenu sets the hub-wide write guard on its dirs, and no test reaches that wiring"
---

Range: `git diff $(git merge-base main HEAD)..HEAD`, since no `_lyx/loom/rework/` round exists and `product.parent` is `main`.
All 24 cards are one commit each and match their card text.

Settled against the focus entries, none of which produced a defect beyond the findings below:

- Slot lifetime: `verifytree.Verify`, `runCardVerify`, `goListLoader.load` and `gatecli.runTest` each release the lease by `defer` on every return path, and each starts its timeout only after the acquire.
  `hubgeom.GateSlots` reads `gate.yaml` inside the `Limits` closure, so every acquire re-reads it.
- Nil pool: every hub-mode wiring passes `hubgeom.GateSlots`: `WebsterGeometry`, `landingDeps`, `loomCLI.wire`, `loomCLI.wireLightweight` and the `lyx webster` hub wiring.
  Nil survives only in `standalonegeom` and in tests.
  The battencli `shedrecipe.Env` runs no gate and needs none.
- Child lifetime: the only change under `internal/verifyrun` is the `env` parameter in `run.go` and its test, and `internal/proc` is untouched.
  The group, `Pdeathsig` and job-object code is in new `internal/gatecli/child_*.go` files.
- Per-card mechanical checks: no card carries a `Rename` or `Delete` group, so those checks have no target.
  The re-signed `Edit` members (`Verify`, `Run`/`run`, `dispatch`, `commandFor`, `NewVerifyGate`, `rerunCardVerifies`, `RenderForkPrompt`, `RenderRecoveryPrompt`, `cardGateCommand`, `ReadRubric`, `burlerRoundProfile`, `verifyWaiting`) have every caller updated.
  `go build ./...` and `go vet -tags integration|tmux|llm` over the packages that touch them are clean.
- Tier placement: the card gate renders `lyx gate test` over the card's packages plus the lint, the batch gate renders `--tags integration` once per batch, and `NewVerifyGate` appends tier 3 only when `checkedPublishFailure` returns a record.
- Publish failure record: `recordFailure` writes on `StatusFailed` with a non-negative exit and no timeout, `clearFailure` runs after the last clean-tree check and before the push, and the rubric renders `PublishFailureNote`, which reads only through `checkedPublishFailure`.
- Hub-wide config: `setModule` and `editOne` both guard, and the menu reaches `editOne`.
  `landing` is `HubWide` and `SeedsFromPrime`, and all three `ReconcileHubWideAt` callers pass the prime.
- Package-scoped runs of the untagged and `integration` tiers over the touched packages pass.

### [BLOCKING] Per-helper arguments test duplicates the CLI scenario

**Location:** internal/gatecli/cli_test.go:47
**Issue:** `TestGoTestArgs_AssemblesDirCapTagsPackagesAndFlagsInOrder` calls the unexported `goTestArgs` directly.
`PATTERN-test-economy` says an unexported helper is tested through the surface that uses it.
`TestGateTest_Scenario`, subtest "a free slot runs go test under the cap and exits with its code" in `internal/gatecli/test_integration_test.go`, already asserts the exact list `test -C <prime> -p 3 -tags integration ./pkg -run X` that the stand-in go binary received, which is the table's second row.
Card 10 names the test, but the pattern binds over the card.
**Fix:** delete the test.
If the packages-only shape needs an exact assertion, add it to the scenario's "-C names a worktree" subtest, which already reads `record.Args`.

### [BLOCKING] New tests skip t.Parallel without naming global state

**Location:** internal/websterengine/cardgate_test.go:189, internal/websterengine/cardverify_test.go:54, internal/landingshed/publish_verify_test.go:352
**Issue:** `PATTERN-test-speed` requires `t.Parallel` unless a comment names the process-global state the test touches.
`TestRenderBatchGate` is pure over `t.TempDir()` and a plan value.
`TestRerunCardVerifies_Slotted` uses a temp-dir pool and wait directory and never mutates the environment.
`TestPublishVerify_FailureRecord` calls `failOnGitHubClient`, which swaps the package variable `NewGitHubClient`, but its doc comment does not say so.
Sibling tests in these files omit it too, which does not exempt a new test.
**Fix:** add `t.Parallel()` to the first two, and to their subtests where each builds its own fixture.
For `TestPublishVerify_FailureRecord`, add a comment naming the swapped `NewGitHubClient`.

### [BLOCKING] Gate refusal rows with no test reaching them

**Location:** internal/gatecli/run.go:57, internal/gatecli/run.go:88
**Issue:** The diff adds a `## gate` section to `contracts/specs/refusal-spec.md` and lists `lyx gate` in its status line, so `PATTERN-refusal-way-forward` binds the module: each new refusal needs a test that reaches it.
`cli_test.go` reaches "no package" and "-C directory missing", and the scenario reaches "slot busy".
Nothing reaches "gate.yaml unreadable or invalid" (the hub-limits error at run.go:57), the "go cannot be started" arm of the raw I/O row (run.go:88), or the "cannot acquire a gate slot" arm.
The behavior is added and uncovered.
**Fix:** add rows to `TestGateTest_Scenario`, or a table-driven refusal test: a hub whose `gate.yaml` holds `slots: 0` expects exit 1 with `lyx config gate` in the error, and a nonexistent `goBinary` expects exit 1 with `put go on PATH`.

### [LOW] Absent gate.yaml detected by error text

**Location:** internal/gateslot/config.go:30
**Issue:** `strings.Contains(err.Error(), "not found")` is the only thing that recognizes an absent file, because `configengine.Load` returns a plain error for it.
`configengine.ErrNotInitialized` documents that callers should not match error text, and the substring also matches any other error containing those words.
The result reads `gate config absent; run "lyx fabric reconcile": config file X not found; run "lyx config reconcile"`, and `gatecli` then appends a third way forward, `lyx config gate`.
**Fix:** `os.Stat` `configengine.ConfigFile(baseDir, "gate")` first and return the absent-file error on `errors.Is(err, os.ErrNotExist)`, then call `Load`.
Name one reconcile command in that error.

### [LOW] Publish failure record fields reach the prompt and the command line past the shape checks

**Location:** internal/loomshed/publishfailure.go:61, internal/loomshed/publishfailure.go:23
**Issue:** The log path check compares only `filepath.Dir`, so any file name in the verify directory passes, a name with a newline and prose included.
`PublishFailureNote` prints it, which contradicts the card's "no free prose reaches the prompt".
`failedTestPackage` accepts a first segment starting with `-`, so a record package of `-c` yields `go test -tags tmux -run '^TestX$' -c`, which compiles and runs nothing.
**Fix:** keep `LogPath` only when `filepath.Base` equals the base of `paths.PublishFailureLog`, and require a package segment to start with a letter or digit.
Add a row to `TestPublishFailure` for each.

### [LOW] Strict-caller list in the config-strictness pattern is stale

**Location:** pattern/PATTERN-config-strictness.md:6
**Issue:** `cmd/lyx/configstrictness_test.go` gains `internal/gateslot` in the strict set, but the pattern's list of strict callers does not name it.
**Fix:** add `gateslot` to the list in the same entry.

### [LOW] Menu guard wiring has no test

**Location:** internal/configcli/configcli.go:507
**Issue:** `runMenu` sets `dirs.hubWriteGuard`.
The guard tests go through `dispatch`, so removing that line fails nothing, and the menu would write a hub-wide module from a pair.
**Fix:** extract the menu's dirs construction so a test can assert the guard is set, or drive `menu` with a guard that refuses and a hub-wide selection.
