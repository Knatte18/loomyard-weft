# Webster round 4 — fixer report

## Fixed

### F1 — whole-plan validate verbs ran the plan gate's type load unslotted

Commit `97013be91`.
The premise held: `planglyph.Validate` called `planGatePass` with `defaultTypesLoader`, and `lyx webster validate` (both whole-plan branches of `scopedValidate`) and `lyx loom validate-plan --require-approved` called it directly instead of through the slotted index.

- `planindex.Index` gains `Validate`, the whole-plan check with the approval gate.
- `planglyph.Validate`'s body moves into `validate(plan, root, loader)`, and the index's `Validate` runs it through the index's own loader, so `NewSlottedIndex` slots it.
- `internal/webstercli/validate.go` calls `c.geom.Index.Validate`, and `internal/loomcli/validate.go` calls `c.env.PlanIndex.Validate`; their doc and help text name the index's `Validate`.
- Test: the slotted-load subtest of `TestGlyphChain_CallerCoverageTyped` becomes a two-row table, the existing plan-gate-pass row plus a row driving `NewSlottedIndex(...).Validate` against a held one-slot pool and comparing it with the unslotted whole-plan answer.
- `newTestCLI` in `internal/webstercli/cli_test.go` now wires the code index the CLI layer wires; `hubGeometryWithIndex` moved there from the integration-tagged `verbs_test.go` so the untagged tier can use it.

### F2 — an unloadable gate.yaml named no way forward at the Go-side gate sites

Commit `00f101500`.
The premise held: `hubgeom.GateSlots` wrapped `gateslot.LoadConfig`'s error with no way forward for anything but the absent file, and the refusal spec had no webster or loom row for it.

- `gateslot.LoadConfig` appends `fix it with "lyx config gate" from the prime` to every failure except the absent file, whose `ErrConfigAbsent` already names `lyx fabric reconcile`.
  It is appended in `LoadConfig` rather than `decodeConfig`, so a parse failure carries it too and `TemplateConfig` does not.
  Every Go-side site that wraps the error inherits the clause.
- `contracts/specs/refusal-spec.md` gains a `gate slot: gate.yaml absent or invalid` row in the webster section (verify gate, `lyx webster verify`, card-verify rerun) and in the loom section (round gate, Publish, Finalize).
- Test: `TestLoadConfig`'s below-1 rows now assert the full error text with the way forward.
  The review suggested extending `TestRerunCardVerifies_Slotted`'s expected text instead, but that row injects its own `Limits` error and never reaches `LoadConfig`, so its text cannot change; the assertion that reaches the new clause is `TestLoadConfig`'s.

## Gate repair

Commit `3863f353c`.
The round's verify gate failed on `TestOverview_RealFilePassesCheck` in `internal/pattern`: `PATTERN.md` was 16559 bytes, over its 16384-byte cap.
Neither fix touched the file. The mid-run merge of `main` grew the overview to 16269 bytes, and this branch's `PATTERN-gate-slot` entry pushed it over.
The entry is shortened to `Running a gate build or test: only in a -p-capped hub slot of internal/gateslot`, which brings the file to 16380 bytes.
The entry no longer lists the sites that take a slot, since such a list goes stale whenever a site is added.
It also drops its rule that no agent starts a raw module-wide `go` run, because the background of `PATTERN-no-denied-recovery` already states it.
`go test ./internal/pattern/ ./cmd/lyx/ ./contracts/...` passes.

## Verification

`go build ./...` passes, and `go vet` under the `integration`, `tmux` and `llm` tags is clean over every changed package.
`go test` and `go test -tags integration` pass over `planglyph`, `webstercli`, `loomcli`, `gateslot`, `gatecli`, `hubgeom`, `websterengine`, `contracts/stencils` and `cmd/lyx`.
`lyx gate test` could not be used: the installed `lyx` has no `gate` command yet, so the runs were package-scoped `go test -p 2`.
`lyx loom lint-comments` passes.

## Disputed

None.

## Deferred

None.
