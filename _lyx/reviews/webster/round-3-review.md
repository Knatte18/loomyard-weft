---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/websterengine/cardgate_test.go:30
    summary: the range adds two gofmt violations, plan :=&planparser.Plan here and a misaligned seed field in configcli_test.go
  - id: F2
    severity: LOW
    class: consistency
    location: internal/fabriccli/fabric.go:326
    summary: the reconcile help says it heals the hub-wide fabric.yaml and board.yaml, while gate.yaml is now hub-wide and the gate refusals name reconcile as the fix for its absence
---

### [LOW] Two gofmt violations in test files the range touches

**Location:** `internal/websterengine/cardgate_test.go:30`, `internal/configcli/configcli_test.go:643`
**Issue:** `gofmt -l` over the range's added or modified Go files lists exactly these two.
`cardgate_test.go:30` reads `plan :=&planparser.Plan{Language: "go"}`, introduced by the F1 fix commit `3e0c3bf33`.
`configcli_test.go:643` has `seed        string` mis-aligned against its sibling fields in the `rows` struct, introduced by card 17 (`681c2135f`); the file was gofmt-clean at the merge base.
**Fix:** run `gofmt -w` on both files.

### [LOW] `lyx fabric reconcile` help omits gate.yaml

**Location:** `internal/fabriccli/fabric.go:326`
**Issue:** The `reconcile` `Long` text says it "heals the hub-wide config files (fabric.yaml and board.yaml)".
Card 2 registers `gate` in `configreg` as `HubWide`, and `configsync.ReconcileHubWideAt` walks every `HubWide` module, so reconcile does write an absent `gate.yaml`.
The `gate.yaml absent` row of `contracts/specs/refusal-spec.md` and the `lyx gate test` envelope both name `lyx fabric reconcile` as the way forward, so an operator following it reads help that does not mention the file.
The range does not touch `internal/fabriccli`, so the help went stale with this change.
**Fix:** name the hub-wide config files generically in the help, or add `gate.yaml` to its list.

## Focus verdicts

### cardgate_test.go parallelism

Not a defect.
Every top-level test in the range's version of `internal/websterengine/cardgate_test.go` (`TestCardGateCommand`, `TestRenderCardGates_OneBulletPerCardWithAStepSubBulletEach`, `TestRenderBatchGate`) calls `t.Parallel()`, and so does every `t.Run` subtest and each row of the two table loops (the loop variable is per-iteration under the module's Go version).
The tests share nothing but their own `t.TempDir()` roots, so no global-state comment is needed.
`go test ./internal/websterengine` passes.

### Absent gate.yaml envelope

Not a defect.
`LoadConfig` in `internal/gateslot/config.go` stats the file and returns `ErrConfigAbsent`; `hubgeom.GateSlots` wraps it with `%w`, so `errors.Is` in `internal/gatecli/run.go:56` matches.
A freshly built binary against this hub, which has no `gate.yaml`, printed `gate test: the hub has no gate limits: load .../gate.yaml: gate config absent; run "lyx fabric reconcile"; way forward: run "lyx fabric reconcile", then re-run the same command`.
That is one command, `lyx fabric reconcile`, and it agrees with the `gate.yaml absent` row of the `## gate` section of `contracts/specs/refusal-spec.md`.
The embedded error text and the trailing clause both name it, which is repetition and not a second way forward.
The integration scenario in `internal/gatecli/test_integration_test.go` ("an absent gate.yaml names lyx fabric reconcile") removes the hub's file and pins the way-forward clause in the JSON envelope; `go test -tags integration ./internal/gatecli` passes.

## Other checks

- `go test` passes for `internal/websterengine`, `internal/gatecli`, `internal/gateslot`, `internal/hubgeom`, `internal/verifytree`, `internal/loomshed`, `internal/commentlint`, `cmd/lyx` and `contracts/stencils`; `go test -tags integration` passes for `internal/gatecli` and `internal/gateslot`.
- The `.dev-bin/lyx` in this worktree was built before commit `063228df9` and still prints the pre-fix absent-file envelope (two reconcile commands and a `lyx config gate` way forward).
  That is a stale build, not a defect in the range; the fresh binary used for the check above was built into the session scratchpad.
- `cmd/lyx/testdata/test-budget.yaml` rows moved for the packages whose top-level test count rose, and the budget test passes.

## Focus departures

None.
