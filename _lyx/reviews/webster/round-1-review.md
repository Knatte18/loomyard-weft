---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/loomcli/wiring.go:439
    summary: "A comment still says every other loader takes the (baseDir, module) shape, which this diff replaced with told dirs"
    fix: "In internal/loomcli/wiring.go, rewrite the comment above the burlerengine.LoadConfig call (the one starting 'burlerengine.LoadConfig takes one argument'). Drop the '(baseDir, module) shape every other loader above takes' contrast and keep only the reason that holds: it is an optional-file loader, so an absent burler.yaml yields a zero Config and a nil error rather than a load failure."
  - id: F2
    severity: LOW
    class: consistency
    location: pattern/PATTERN-config-layers.md:21
    summary: "The docs say no cleanup ever deletes a local override file, yet Seed-Child removes the local files of modules the run settings do not name, and reconcile deletes a local file a prune empties"
    fix: "In pattern/PATTERN-config-layers.md (Local files section) and in the configengine package doc (internal/configengine/doc.go, the sentence 'A local override file is never disposable: no cleanup deletes it.'), restate the rule so it matches the code. Say that no cleanup deletes a local override, and that the only removals are Seed-Child rewriting the pair's local layer to equal the task's run settings, and reconcile deleting a local file whose retired keys it pruned to nothing."
  - id: F3
    severity: LOW
    class: consistency
    location: internal/boardengine/runlock_test.go:180
    summary: "TestRunStatusProducer repeats the done, hand-set word and nil rows that the run status form subtest already pins through IsRunStatus, which delegates to RunStatusProducer"
    fix: "Delete TestRunStatusProducer. In the 'run status form' subtest of the test that holds it (internal/boardengine/runlock_test.go, around line 150), add one assertion that boardengine.RunStatusProducer(strPtr(\"running · Seed-Child\")) returns \"Seed-Child\", true. Then lower the internal/boardengine row in cmd/lyx/testdata/test-budget.yaml from 77 back to 76."
---

### [LOW] Stale loader-shape comment in loomcli wiring

**Location:** internal/loomcli/wiring.go:439
**Issue:** The comment above `burlerengine.LoadConfig` contrasts it with "the (baseDir, module) shape every other loader above takes".
This diff moved every module loader onto told `configengine.Dirs`, so no loader takes `baseDir` any more and the contrast is false.

### [LOW] "Never disposable" local-file rule is stated more strongly than the code keeps it

**Location:** pattern/PATTERN-config-layers.md:21
**Issue:** `PATTERN-config-layers` and the `configengine` package doc say a local override file is never disposable and that no cleanup deletes it.
`battenshed.seedChildProducer.writeRunSettings` removes the local file of every registered module outside the task's run settings, which card 21 prescribes.
`configsync.reconcileSparseFile` also deletes a local file that a prune empties.
Both removals are intended, but the invariant as written is absolute and a reader will take it as a prohibition these two paths break.

### [LOW] New top-level test duplicates an existing subtest's coverage

**Location:** internal/boardengine/runlock_test.go:180
**Issue:** `PATTERN-test-economy` and the rubric flag a new test that duplicates an existing one.
The "run status form" subtest of the run-lock test already pins `IsRunStatus` for nil, `done`, a hand-set word and the run-status forms, and `IsRunStatus` is now `RunStatusProducer`'s thin wrapper.
`TestRunStatusProducer` adds one new fact, the producer value, in a separate top-level test that also costs a budget row.

## Rejected

- The classification in every module's `ConfigSpec().Categories` matches the plan's per-key table, and `TestModules_EveryTemplateKeyClassified` fails on an unclassified template key and on a Categories entry naming no template path.
- Every local-layer entry point goes through `configengine.ValidateLocal`: `Layers` for all loaders, `Set` and `Edit` for `lyx config --layer local`, `configreg.PrepareRunSettings` for board writes and Seed-Child. `LocalKeys` and `LocalDenied` are derived from the same `Categories`.
- `configsync`, `hubreconcile` and `configcli` read layer files directly for reconcile, snapshot and print, with paths formed by `configengine`. `PATTERN-config-layers` governs resolving and stacking, which only `configengine` does, so this is not flagged.
- The `Dirs{Anchor: res.StateDir}` loads in `burlercli` and `webstercli` are the standalone paths, where no board dir exists, and are correct.
- `CloneAndWire` returns the walk error rather than swallowing it, as card 15 specifies, with the clone left on disk.
- An open map is carried whole from the highest layer, so a sparse `--set` of one entry replaces the lower layers' entries; the decision record and card 1 choose this, so it is not a defect.
- `go test` over the changed config, board, batten and agentname packages and `go test -tags integration` over `hubreconcile`, `battencli`, `fabriccli` and `configcli` pass, and the `test-budget.yaml` rows match the new top-level test counts.
