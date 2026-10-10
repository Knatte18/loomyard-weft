---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/gatecli/prebuild.go:36
    summary: The prebuild go build of lyx runs without the slot's -p cap, which PATTERN-gate-slot requires of every gate build
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/hubforge/template.go:211
    summary: CopyHub rewrites the tracked .lyx-warp record so a copy is never a clean hub, and six tests were put back on NewHub against the fixture-classification decision with the bound recorded nowhere
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/testkit/tmuxkit/tmuxkit.go:71
    summary: Main nests a second kit directory inside an inherited one so a re-exec'd helper overflows the tmux socket path under the harness TMPDIR, and the shipped after-report records that package as FAIL unfixed
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/gitrepo/gitrepo.go:109
    summary: Repo.Toplevel is a second worktree-root resolver outside lyxcwd with no production caller, against PATTERN-cwd-resolution and the no-unused-code rule
  - id: F5
    severity: LOW
    class: scope
    location: cmd/testtiming/prebuild.go:20
    summary: The harness prebuildLyx has no test for its strip, build, no-cmd/lyx skip or build-failure branches
  - id: F6
    severity: LOW
    class: scope
    location: internal/gatecli/run.go:89
    summary: The tagged-run build-failure refusal (cannot build lyx, exit 1 before go test) has no test
  - id: F7
    severity: LOW
    class: consistency
    location: internal/loomcli/smoke_resumearming_test.go:46
    summary: TestLoomResumeArmsAfterReconcile asserts the reed session is absent instead of up as card 9 specifies, and the absence check passes on any Status error
  - id: F8
    severity: LOW
    class: consistency
    location: internal/testkit/tmuxkit/tmuxkit_integration_test.go:195
    summary: TestPackageServer_OneKeyPerBinary and TestAfterRun_LeftoverProcesses skip t.Parallel without a comment naming the global state, against PATTERN-test-speed
  - id: F9
    severity: NIT
    class: consistency
    location: internal/testkit/tmuxkit/tmuxkit.go:5
    summary: The package comment's PackageServer sentence is ungrammatical and leaves Both ambiguous, and the PATTERN-testkit tmuxkit bullet omits the widening to starting servers under the kit config
---

Range reviewed: no live rework round exists, so `git diff $(git merge-base main HEAD)..HEAD` (`product.parent` is `main`).
Evidence gathered by driving the substrate through `lyx gate test`: the untagged tier of the changed packages, the `integration` tier of `hubforge`, `gitrepo`, `lyxcwd`, `gitexec`, `gitkit`, `gatecli`, `verifytree`, `lyxbin`, `cmd/lyx`, `cmd/testtiming`, `configcli`, `fabriccli`, `hubgeom` and `fabricengine`, and the `tmux` tier of `tmuxkit`, `reedengine`, `reedcli`, `pairteardown`, `webstercli`, `standalonegeom` and `loomcli` all pass.
`PATTERN.md` is 16168 bytes, under the 16200 the decision record sets.
The surviving `rev-parse` spawns in production code are exactly the list in the `rev-parse-survivors` decision.

### [BLOCKING] The prebuild `go build` runs outside the slot's `-p` cap

**Location:** `internal/gatecli/prebuild.go:36`, `cmd/testtiming/prebuild.go:37`
**Issue:** `PATTERN-gate-slot` requires a gate build or test to run in a `-p`-capped hub slot.
`runTest` caps `go test` with `-p <parallel>` in `goTestArgs` and gives it `lease.Env`, which appends the slot's `-p` to `GOFLAGS`.
`prebuildLyx` runs `go build -C <root> -o <bin> ./cmd/lyx` with neither: no `-p` argument, and `cmd.Env` is unset so the child inherits the process environment, not `lease.Env`'s `GOFLAGS`.
A tagged gate run therefore compiles `lyx`, with its cgo tree-sitter grammars, at the machine's full parallelism while holding a slot meant to bound load.
`cmd/testtiming/prebuild.go` does the same with `go` from PATH.
Card 13 prescribes the exact command line, and `TestGateTest_Scenario` pins `build -C <prime> -o <bin> ./cmd/lyx`, so the plan and the test both encode the uncapped form; the constraint outranks the card.
**Fix:** Pass the run's `parallel` into `prebuildLyx` and add `-p <parallel>` to the build arguments (give `cmd/testtiming` the cap it already resolves for its `go test` children), then update the scenario step's expected build arguments.

### [MEDIUM] `CopyHub` copies are never clean hubs, and the exception is unrecorded

**Location:** `internal/hubforge/template.go:211` (`relocateHub`)
**Issue:** `relocateHub` rewrites the template root in every text file, including the tracked `.lyx-warp` record on the records side, whose committed content still names the template's bare repository.
Every copy's board checkout is therefore dirty against its own HEAD.
Three fix commits found this and put six cells back on `NewHub` with a one-line test comment: `TestCloneHubReset`, the `clean` state in `TestStates`, `CloneHubReset/RealHub` in `TestVerbCases_CleanState`, `TestStencilSeeding_HubScenario`, `TestConfigOverRealHub` and `TestRunCLI_HubMutationScenario`.
That departs from the `hub-fixture-classification` decision, which lists only `TestSpawnAnchoredScenario` and the `wts/some-task` case as fresh and says every other writer copies.
`TestCopyHub` asserts a clean `fabric status` but never a clean records checkout, so the defect has no test, and neither `hubforge`'s `doc.go`, `CopyHub`'s doc comment nor `PATTERN-hubforge-fixtures.md` states that a copy differs from a fresh hub.
The next writer that asserts a clean tree will rediscover it.
**Fix:** Make the copy faithful by committing the rewritten record in the copy's records checkout (and pushing it to the copy's bare), with a `TestCopyHub` assertion that the records checkout is clean; then return the six cells to `CopyHub`.
If the rewrite stays, state the bound in `CopyHub`'s doc comment, `doc.go` and `PATTERN-hubforge-fixtures.md`, and amend the classification so the six cells are fresh by rule.

### [MEDIUM] Nested kit directory overflows the socket path; the shipped report records a FAIL

**Location:** `internal/testkit/tmuxkit/tmuxkit.go:71`, `docs/benchmarks/test-suite-weight.md:615`
**Issue:** `Main` always creates its directory under `os.MkdirTemp("")` and points `TMPDIR` at it.
A re-exec'd helper of the same test binary, such as `TestPushCoalesced`'s children, inherits that `TMPDIR` and calls `Main` again, so its kit directory nests inside the parent's.
Under the `-resources` harness, whose per-package `TMPDIR` is already one level deep, the socket path passes the 107-byte `sun_path` limit and the package fails.
The `## After` section of the report states this and ships the `internal/gitrepo` tier-2 row marked `FAIL`; the harness card 2 delivers cannot measure a package the diff's own kit breaks.
The same section records a `sleep 300` leftover from `internal/reedcli` in tier 3; it did not reproduce in my `lyx gate test --tags tmux ./internal/reedcli` run, so it needs the reedcli author's attention rather than a fix here.
**Fix:** Have `Main` reuse an inherited kit directory instead of nesting (for example when `TMUX_TMPDIR` already names a directory carrying the `lyx` prefix and the process is a helper), or root the kit's directory at a short fixed base; then regenerate the `## After` tables so no row is `FAIL` for a harness-induced reason.

### [MEDIUM] `Repo.Toplevel` duplicates `lyxcwd` and nothing calls it

**Location:** `internal/gitrepo/gitrepo.go:109`
**Issue:** `PATTERN-cwd-resolution` says only `internal/lyxcwd` resolves a worktree root.
`Toplevel` walks up to the worktree root through `dotgit.FindRoot`, which is that resolution under another name; `PATTERN-gitrepo-client-boundary` was amended to admit the `dotgit` geometry read but `PATTERN-cwd-resolution` was not.
It has no production caller, only the oracle comparison in `parity_test.go`, so it is also exported dead code with an oracle method and a parity row maintained for it.
**Fix:** Delete `Repo.Toplevel`, `gitoracle.Toplevel`, its parity row, its `doc.go` mention and the oracle allowance in `internal/lyxcwd/enforcement_test.go`.
If a caller is wanted, route it through `lyxcwd` and say so in `PATTERN-cwd-resolution`.

### [LOW] Harness `prebuildLyx` has no coverage

**Location:** `cmd/testtiming/prebuild.go:20`
**Issue:** The function strips an inherited `LYX_PREBUILT_LYX` on an untagged run, builds and exports the binary on a tagged run, skips the build when the root has no `cmd/lyx`, and returns the build output on failure.
Three modes call it (`run`, `runRedundancy`, `runResources`) and no test reaches any branch; the gate's equivalent has scenario steps for the first three.
**Fix:** Add rows to a table-driven `TestPrebuildLyx` over a temporary root: untagged with an inherited value, tagged without `cmd/lyx`, and tagged with a `cmd/lyx` that does not build.

### [LOW] Tagged-run build failure in the gate is untested

**Location:** `internal/gatecli/run.go:89`
**Issue:** Card 13 states that a build failure is reported on the envelope with the build's output and exit code 1 before `go test` runs.
`TestGateTest_Scenario` covers the tagged build, the untagged strip and the no-`cmd/lyx` skip, but no step makes the build fail, so the refusal message and the "no `go test` after a failed build" ordering are unpinned.
**Fix:** Add a scenario step whose stand-in `build` invocation exits non-zero and assert the envelope text, exit code 1 and that no `test` record follows.

### [LOW] Resume-arming test asserts the opposite of card 9

**Location:** `internal/loomcli/smoke_resumearming_test.go:46`
**Issue:** Card 9 says `TestLoomResumeArmsAfterReconcile` asserts the stamp exists and the pair's session is up on the hub's key.
The test asserts the stamp, the no-live-driver refusal, and that reed `Status()` returns an error, which it reads as no session.
Any `Status` failure, such as a config or tmux error, satisfies that last assertion, so it cannot tell "resume brought nothing up" from "the probe broke", and the name says "Arms" while the body checks that nothing armed.
**Fix:** Assert the specific no-session outcome `Status` reports, or drop the reed assertion if the refusal already pins the behavior, and rename the test to what it checks.

### [LOW] New tmuxkit tests skip `t.Parallel` without naming global state

**Location:** `internal/testkit/tmuxkit/tmuxkit_integration_test.go:195`, `internal/testkit/tmuxkit/tmuxkit_integration_test.go:213`
**Issue:** `PATTERN-test-speed` requires `t.Parallel` unless the test touches process-global state and a comment names it.
`TestPackageServer_OneKeyPerBinary` uses the package-level server and `TestAfterRun_LeftoverProcesses` works on its own temporary directory; neither calls `t.Parallel` and neither carries a comment.
`TestSocket_StartsHermeticNonLoginServer` in the same file shows the expected form.
**Fix:** Add `t.Parallel` to `TestAfterRun_LeftoverProcesses`, and a comment naming the package server as shared state on `TestPackageServer_OneKeyPerBinary`.

### [NIT] Package comment wording and bound drift

**Location:** `internal/testkit/tmuxkit/tmuxkit.go:5`, `pattern/PATTERN-testkit.md`
**Issue:** The `PackageServer` sentence in the package comment ("hands every test of the package that names its own sessions, kills no server and asserts nothing about the whole session set one shared server, which Main's sweep kills") is not a sentence, and the following "Both first start a hermetic server" no longer clearly refers to `Socket` and `KillOnCleanup`.
The package comment widens the exemption to "starting servers under the kit's own config", while the `PATTERN-testkit.md` tmuxkit bullet widens it only to the `/proc` scan.
**Fix:** Rewrite the `PackageServer` sentence and name `Socket` and `KillOnCleanup` in the next one; add the server-start bound to the `PATTERN-testkit.md` bullet.

## Focus verdicts

- Per-card mechanical check: the plan carries no `Rename`, `Delete` or `Move` group, so there is no per-type check to run; cards 17 to 22 are `Edit`-only, and every file they list changed.
  Cards 28 to 30 leave exactly the survivors the `rev-parse-survivors` decision names.
- Test economy: no duplicate found among cards 5 to 10, 16, 23 to 26 and 31 to 32.
  `dotgit_test.go` and the `lyxcwd` scenario steps overlap in layouts, but each pins its own package's public surface and the card asks for both.
- Behavior coverage: the reed stale-holder probe (pid and grace assertions in `TestEnsureSession_BootedTrueOnColdSessionFalseOnWarm`), the leftover failure (`TestAfterRun_LeftoverProcesses`), the socket check (`TestCheckSockets`), the `GIT_DIR` and `GIT_WORK_TREE` drop (`TestRun` row), the gate strip and the webster and verify strips and the raw-go deny row are covered; the harness strip (F5) and the gate build failure (F6) are not.
- Decision conformance: the classification departs from the decision for the six cells in F2; the survivors list conforms.
- Comment conventions: no finding beyond F9; no line-break finding is raised, per `PATTERN-comment-line-breaks`.
