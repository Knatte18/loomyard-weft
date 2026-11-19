```yaml
lens: row-validity
outcome: reported
```

Lens: row-validity. Each planned test row was checked for whether it reaches its branch, whether it fails when the code is wrong, and whether it should be new or a row of an existing test.

**MEDIUM**

- **Card 17, `TestRemovePair_SandboxClear`: the "worktree holds the root" branch cannot be reached through `fakeSubstrates`.**
  - The card puts the sandbox-root existence check inside `RemovePair` itself.
  - `RemovePair` finds the worktree's path from the real prime: `taskAnchor` calls `fabricengine.WorktreePath(t.prime, slug)` and `lymcwd.NamedWorktree(t.prime.HubPath, slug)` (`internal/pairteardown/teardown.go:644-650`).
  - A `fakeSubstrates`-driven `Teardown` has no on-disk worktree where a `.lym/harden-sandbox` could be planted, and the card adds no fake field for the path or the existence check.
  - Two fixes would work: have the seam take the slug and report whether it cleared anything (`clearSandbox(slug) (bool, error)`), or add a stat seam to `fakeSubstrates`.
  - The new test also belongs as rows of `TestTeardown_CallSequence` (`teardown_test.go:353`), which pins the step order such as `archive,remove,…`.
- **Card 6, `TestLiveGate`: the "package without `tmuxkit.Main` is named outside the cover" row cannot be built as written.**
  - `LiveDeps` holds the run directory, `Round`, the packages, `llm_driving`, `concurrency` and `Run`, but no worktree root.
  - The relative `packages` therefore cannot be resolved to read their `_test.go` files. The gate would read relative to the process's working directory, which in production is the shed driver's.
  - Fix: add the worktree root to `LiveDeps`, or a told `CoversStrays(dir) bool`.
- **Card 10, `OnHalt` in `escalateWith`: the branch for other escalation causes is unspecified and untested.**
  - `escalateWith` serves every cause: circling, budget, hard-cap, decision and stalemate (`internal/shedadapters/escalation.go:24-30`).
  - The card calls `OnHalt` for all of them, so a decision or stalemate escalation would also reap the sandbox mid-campaign.
  - It also leaves open whether the "escalation already recorded returns its Awaiting" replay re-enters `escalateWith` and reaps again on every re-step (`bouncer_escalation_test.go`, row "an escalation already recorded returns its Awaiting").
  - `TestBouncer_HaltRunsTheHaltSeam` covers only circling and hard-cap. It needs a row pinning that a decision or stalemate cause, and a replay, do not call `OnHalt` (or do, if that is intended).
- **Card 16, `TestSandboxVerbs`: the "deploy seam is a fake here" premise has no named injection point.**
  - `hardencli.Command()` takes no dependencies, and the card lists no package-level seam, `export_test` hook or deps struct through which the test replaces `Deploy`.
  - As written, `new` in a tier1 test would run `go run ./tools/deploy -dev` whenever the revision differs, which it always does in a fixture.
  - Fix: name the seam, for example a deps-taking `newCommand(deps)` behind `Command()`.

**LOW**

- **Card 1, `TestRunTierCopies`: wrong tier and fixture, and some rows sit at the wrong level.**
  - The card says tier1 in a hub fixture "as `TestRunTierProcessCap` drives one", but `TestRunTierProcessCap` is tier0 with stand-in factories and no hub (`internal/gatecli/tierrun_test.go:138-139`).
  - The two key refusals live in `gateKeyRefusal(cfg gateslot.Config, request testRequest)` (`run.go:430`), a pure function with no existing test. They are tier0 rows over a hand-built `Config`, not hub-fixture cases.
- **Card 15, `TestFinalize_ReapsBeforeLanding` should be rows of existing tests.**
  - The failing-reap halt belongs in `TestFinalize_StuckReasons` (`finalize_test.go:670`).
  - The reap-before-merge order belongs in `TestFinalize_MergeMarkPushOrderAndVerdict` (`finalize_test.go:171`), which already pins Finalize's call order.
- **Card 4: the "second `HardenFix` row is refused as two build rows" row repeats a behaviour an existing row reaches.**
  - It goes through the same `onlyRowOf` branch as the existing row "two build rows" (`internal/shedbuild/build_test.go:201-204`).
  - Only the engine-choice text changes, and the existing rows' strings already pick that up.
- **Card 14: `TestWire_EverySeamFilled` itself does not change.**
  - The test checks against the `intentionallyNil` map (`internal/loomcli/wiring_seams_test.go:194-204`), so marking `Env.Harden` and `Env.PreflightHarden` as intentionally nil is an edit to that var.
  - The card lists the test function instead of `internal/loomcli#intentionallyNil`.
- **Card 2, `TestHubSetsGate`: the "unreadable hub file returns its error" row has no listed way to make the file unreadable.**
  - PATTERN-test-isolation requires `permkit.Chmod` for that, and the card does not list it under Uses.

Tool calls:
1. Grepped `internal/gatecli`, `loomcli`, `landingshed`, `pairteardown` and `shedadapters` tests for the named functions' existence and tiers. My context lacked the test files.
2. Read `TestWire_EverySeamFilled`, the `TestBouncer_AutoAccept` table, `gateKeyRefusal` test usage and `pairteardown`'s worktree-path derivation. My context lacked those bodies.
