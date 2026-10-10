# Webster round 1 — fixer report

Every finding was validated against the code and the plan, and all nine are fixed, each in its own commit.
After the last commit, `lyx gate test ./...` and `lyx gate test --tags integration ./...` are both green.
Each fix's own packages also passed in their `tmux` tier where the fix touched it (`tmuxkit` and `loomcli`).

## Fixed

### F1 — prebuild `go build` runs outside the slot's `-p` cap (BLOCKING)

Commit `7b5fcd59b`.
`gatecli.prebuildLyx` takes the run's `parallel` and builds with `go build -C <root> -p <cap> -o <bin> ./cmd/lyx`.
The `gatecli` doc's `Verb` section names the cap.
`cmd/testtiming`'s `prebuildLyx` now takes the root, the base environment and a cap.
`-resources` takes its gate slot first and builds inside it, under the slot's cap and with the lease's environment.
The unslotted `run` and `-redundancy` modes pass a zero cap, because their own `go test` runs uncapped.
`TestGateTest_Scenario`'s tagged step now expects `-p 3` in the build arguments.

### F2 — `CopyHub` copies are never clean hubs (MEDIUM)

Commit `9e15e5ed6`.
A throwaway probe showed that in every shape only `_board`'s `.lyx-warp` is dirty.
The board's `main` is a single commit with no descendants.
`CopyHub` now amends the relocated binding into that commit and force-pushes it to the copy's own records bare.
That leaves the copy with the same one-commit board history a fresh hub has.
`TestCopyHub` asserts that the board and records checkouts are clean and that the board is not ahead of its upstream.
All six cells moved back from `NewHub` to `CopyHub`, as the `hub-fixture-classification` decision has them: `TestCloneHubReset`, the `clean` state in `TestStates`, `CloneHubReset/RealHub` in `TestVerbCases_CleanState`, `TestStencilSeeding_HubScenario`, `TestConfigOverRealHub` and `TestRunCLI_HubMutationScenario`.
All six pass on copies.
`CopyHub`'s doc comment and `hubforge`'s `doc.go` state the step.
Cost: three fixture git spawns per `CopyHub` (add, commit, push).
The regenerated tier 2 census shows them as roughly 750 more each of `commit`, `add` and `push`.

### F3 — nested kit directory overflows the socket path; report records a FAIL (MEDIUM)

Commits `074b0aff2` (kit) and `9408274ed` (report).
`tmuxkit.Main` creates its directory under `kitBase()`.
A process that inherited another `Main`'s directory as both `TMPDIR` and `TMUX_TMPDIR` gets that directory's parent, so a re-exec'd helper's directory sits beside its parent's at the same depth, never inside it.
`TestSetEnv` gains the regression assertions: `kitBase()` is `""` before `setEnv`, and the parent of the kit directory after it.
The package comment and `PATTERN-test-isolation.md` record the behaviour.
`go run ./cmd/testtiming -resources -tags integration -pkg ./internal/gitrepo` now passes under the harness.
All three tiers were rerun on the fixed tree (`9e15e5ed6`), and the `## After` tables, prose and Comparison were regenerated from that run.
No row is `FAIL`, and the reedcli `sleep 300` leftover did not recur.

### F4 — `Repo.Toplevel` duplicates `lyxcwd` and nothing calls it (MEDIUM)

Commit `8cdeaffc1`.
`PATTERN-cwd-resolution` routes every worktree-root query through `lyxcwd`, and `Toplevel` had no production caller.
Deleted: `Repo.Toplevel`, `gitoracle.Toplevel`, its parity row, the `lyxcwd` scan allowance that existed only for that oracle, and the `doc.go` mention.
The doc now says the worktree root is `lyxcwd`'s alone.
This departs from card 25's `Create:` list; per the instructions, the constraint outranks the card.

### F5 — harness `prebuildLyx` has no coverage (LOW)

Commit `f895df0cc`.
The new integration-tagged `cmd/testtiming/prebuild_integration_test.go` is a `TestPrebuildLyx` table with four rows.
Its rows cover the untagged strip, the tagged no-`cmd/lyx` skip, the build and export with its cleanup, and the build-failure error carrying the build output.
It builds a throwaway module.
The package gains a `testmain_test.go` that runs `tmuxkit.Main`, which `PATTERN-test-isolation` requires once a package has an integration file.
`cmd/testtiming`'s budget row rises from 20 to 21.

### F6 — tagged-run build failure in the gate is untested (LOW)

Commit `b57d217b8`.
The stand-in's build invocation now exits with `GATECLI_TEST_BUILD_EXIT`.
A new step in `TestGateTest_Scenario` asserts exit 1, the `cannot build lyx for the tagged run` refusal, and that no `test` invocation follows the build.
The commit also restores gofmt spacing on F1's build-args line.

### F7 — resume-arming test asserts the opposite of card 9 (LOW)

Commit `3703af692`.
The test now requires `errors.Is(err, reedengine.ErrNoSession)` from reed's `Status`, not any error.
It is renamed `TestLoomResumeReconcilesThenRefusesWithoutLiveDriver`.
Its header and the integration test's cross-reference follow the rename.
Card 9's `Verify:` line still names the old test, and that line lives in the plan, which this round may not touch.

### F8 — new tmuxkit tests skip `t.Parallel` without naming global state (LOW)

Commit `efb5c4931`.
`TestPackageServer_OneKeyPerBinary` and `TestAfterRun_LeftoverProcesses` now call `t.Parallel`.
Neither touches process-global state: `PackageServer` is built to be shared behind a `sync.Once`, and the leftover scan works on its own directory.
This is stricter than the review's suggested comment for the first test.

### F9 — package comment wording and bound drift (NIT)

Commit `8390f3a37`.
The `PackageServer` sentence now parses, and the next sentence names `Socket`, `KillOnCleanup` and `PackageServer` explicitly.
`PATTERN-testkit.md`'s tmuxkit bullet admits starting servers under the kit's own config.

## Disputed

None.

## Deferred

None.

## Notes for the loop

- `internal/fabricengine/pull.go`, changed by card 30, ends with a trailing blank line that `gofmt -l` flags.
  The review raised no finding for it, so it is untouched.
- The `## After` yaml cites `9e15e5ed6`, the commit all three regenerated tiers ran on.
  Only `docs/` changed after it.
