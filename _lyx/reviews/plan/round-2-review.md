---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/plan/06-fabric-unwire-apply.md
    summary: Card 6 names only the fabricengine doc.go Unwire paragraph, leaving the Unwire, runUnwire and UnwireVerbResult field doc comments and both unwire.go file headers stating that unwire always removes
  - id: F2
    severity: LOW
    class: scope
    location: _lyx/plan/05-reconcile-floor-migrate.md
    summary: Card 5 gives loom a Migrate hook but leaves TestConfigReconcileInGitRepo asserting that the loom entry omits migrated because loom has no migration, which becomes false
---

### [LOW] Card 6 leaves the unwire doc comments and file headers describing removal only

**Location:** `_lyx/plan/06-fabric-unwire-apply.md`, the `Docs:` sentence of `**Intent:**`.

**Issue:** The card's doc work names only `internal/fabricengine/doc.go`'s `Unwire` paragraph.
After the change, several doc comments on symbols the card itself edits would contradict the new dry-run mode:

- `UnwireVerbResult.JunctionsRemoved` (`internal/fabricengine/unwire.go`) says it "lists the Name of each warp junction that was actually present and removed". A dry run fills it with junctions that were not removed.
- `GitExclude`'s trailing `"reverted" or "unchanged"` note doesn't say that a dry run reports `reverted` for a line it would strip and leaves the line in place.
- The `Unwire` doc comment opens "Unwire reverses every warp junction wired for the worktree at cwd", with no mention of the `apply` parameter.
- `runUnwire`'s doc (`internal/fabriccli/unwire.go`) says it executes the subcommand "removing every on-disk fabric junction for this worktree".
- The file header of `internal/fabricengine/unwire.go` says "Unwire always removes every fabric junction present on disk", and `internal/fabriccli/unwire.go`'s header calls the handler "a per-warp-worktree full deactivation".

Because the Intent lists its doc edits explicitly, an executor following it may leave these comments stale.
Each of them then states a removal the dry run never makes.

**Fix:** Extend card 6's `Docs:` sentence so the `Unwire` doc comment names `apply`, the `JunctionsRemoved` and `GitExclude` field docs state what each reports on a dry run, `runUnwire`'s doc covers the dry-run default, and both `unwire.go` file headers say that without `--apply` the verb only reports.
No target changes are needed: `Unwire`, `UnwireVerbResult` and `runUnwire` are already Edit targets, and the headers sit in their files.

### [LOW] Card 5 leaves a reconcile CLI assertion that rests on loom having no migration

**Location:** `_lyx/plan/05-reconcile-floor-migrate.md`. The test is `internal/configcli/reconcile_integration_test.go`'s `TestConfigReconcileInGitRepo`, the dry-run step.

**Issue:** The dry-run step asserts `reconcileModule(t, result, "loom")["migrated"]` is absent, with the message "loom module entry carries migrated; want it omitted without a migration".
The loom row is the test's example of a module with no `Migrate` hook.
Card 5 registers `loomengine.MigrateConfig` on the `loom` row of `configreg.Modules`.
The assertion still passes, because the seeded `loom.yaml` has no `step_idle_timeout_min` key and the hook returns no line.
But its message becomes false, and the assertion silently changes from "a module without a hook omits `migrated`" to "a hook with nothing to rewrite omits `migrated`".
The card doesn't list the test, so nothing makes the executor revisit it.

**Fix:** List `internal/configcli#TestConfigReconcileInGitRepo` under card 5's `**Edit:**`.
Retarget that assertion to a module that still has no hook, such as `reed`, whose entry the same step already reports, and keep the message accurate.
`internal/configcli` is already covered by the plan's `go test -tags integration ./...`.

## Focus outcomes

- **Card 6: does any new fabriccli test duplicate `TestRunCLI_HubMutationScenario` or the engine tests?** Not a defect.
  Card 6 plans no new top-level test in `internal/fabriccli`, no new test file and no test-budget raise.
  Its one addition is a step inside `TestRunCLI_HubMutationScenario`, which today has no unwire step.
  That step pins only the CLI's own behaviour: `--apply` parsing and the envelope's `applied` key, which no engine test can reach.
  The junction-state and exclude assertions stay in the extended engine tests (`TestUnwire_RemovesOnDiskJunctionsIncludingStale`, `TestUnwire_NeverWiredWarpIsIdempotentNoOp`).
  The step's empty-`mutations` check overlaps the engine test's no-mutation assertion only as one extra assertion, not as a redundant test.
  I checked the card's list of seven `Unwire` callers against the tree: it is complete, covering five tests in `unwire_test.go`, `TestUnwire_PreservesUserSymlinkAtAnchor` and `TestUnwire_LeavesWarpBindingInPlace`.
- **Card 5: is any value shape pinned by more than one planned test?** Not a defect.
  `floor_test.go`'s one table pins every shape once: below, at, above, absent, null, env marker, non-integer, comments preserved and unparseable.
  The three new `TestReconcileAll` rows each use a below-floor value only to reach a different module's hook wiring, its registration and its floor constant.
  No shape is pinned twice for the same unit.
  `internal/yamlengine` sits at its budget row (12) with no `RaiseIntFloor` surface today, so the new test and the one-row raise are justified.
- **Cards 5 and 6: are the sandbox suite sites and doc files that the changed behaviour touches all named?**
  Card 6 names every `lyx fabric unwire` site in `tools/sandbox`: S6's operating-model sentence, goal, durability note and watch line in the core suite, and F5, F11 and F15 in the fabric suite.
  It also names the `fabricengine/doc.go` `Unwire` paragraph.
  The doc comments and file headers outside that paragraph are F1.
  `internal/fabriccli/doc.go` has no unwire section today, and the engine paragraph is the existing home for unwire's contract, so I raise no finding for it.
  Card 5 names `yamlengine`'s, `shuttleengine`'s, `orchengine`'s and `configcli`'s `doc.go`, and correctly notes that `loomengine` has no `doc.go`.
  No sandbox suite asserts reconcile's `migrated` output or a floor value, so card 5 has no sandbox site to name.
  The one reconcile site, S4, expects a clean result after a `--set` and is unaffected.
