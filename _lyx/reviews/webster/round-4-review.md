---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: internal/planglyph/planglyph.go:91
    summary: planglyph.Validate runs the plan gate type load through the unslotted default loader, and lyx webster validate and lyx loom validate-plan --require-approved both call it
  - id: F2
    severity: MEDIUM
    class: consistency
    location: internal/verifytree/verify.go:238
    summary: A gate.yaml that cannot be loaded halts the Go-side gate sites with an error that names no way forward and has no refusal-spec row, where lyx gate test names one
---

Range: `product.parent` is `main` and no `_lyx/loom/rework/` round exists, so the range is `git diff $(git merge-base main HEAD)..HEAD` (36 commits, merge base `10df04b02`).
No focus was set for this round.
I read the production diff in full and the plan cards against it.
`go test` (tier 1) and `go test -tags integration` over every changed package pass, and `go vet` under the `integration`, `tmux` and `llm` tags over every changed package is clean.
The plan carries no Rename or Delete groups, so no type-specific mechanical check applies; every signature change (`verifytree.Verify`, `verifyrun.Run`, `ReadRubric`, `NewVerifyGate`, `RenderForkPrompt`, `RenderRecoveryPrompt`, `rerunCardVerifies`, `dispatch`) has all callers updated and compiling.
I could not run `lyx gate test ./...` against this hub, because the hub's `gate.yaml` is absent until the next reconcile; `lyx gate test` refused with its own `lyx fabric reconcile` way forward, which is the designed behavior.

### [BLOCKING] Whole-plan validate verbs run the plan gate's type load outside a gate slot

**Location:** `internal/planglyph/planglyph.go:91`, reached from `internal/webstercli/validate.go:108`, `internal/webstercli/validate.go:121` and `internal/loomcli/validate.go:189`

**Issue:** `PATTERN-gate-slot` requires the plan gate's type load to go through a slot of the hub pool.
Card 9 slots it by giving the index a slotted loader and states that both `lyx loom validate-plan` and the plan gate "reach it through the index loom wires".
Three call sites do not go through the index.
`planglyph.Validate` calls `planGatePass(plan, worktreeRoot, defaultTypesLoader)`, and `defaultTypesLoader` is the unslotted `goListLoader`, so its `packages.Load` (go list with export data, which compiles dependencies) runs with no slot and no `-p` cap.
`lyx webster validate` reaches it through `scopedValidate` before any batch has begun, and `lyx loom validate-plan --require-approved` reaches it directly.
The default `lyx loom validate-plan` and `--rework` branches use the slotted `c.env.PlanIndex`, which is why the tests that exist pass.
A webster master or an operator running either verb on a busy hub starts an extra build outside the configured slot count, which is the exact state the plan exists to remove.

**Fix:** Route the three call sites through the wired slotted loader.
Either add a loader-taking whole-plan form of `Validate` (as `validateFormat` is for `ValidateFormat`) and call it with the index's loader, or expose the whole-plan check on `planindex.Index` and call `geom.Index` / `c.env.PlanIndex`.
Add one row to the existing slotted-load test (`internal/planglyph/glyphchain_integration_test.go`) showing the whole-plan form waits on a held one-slot pool.

### [MEDIUM] An unloadable gate.yaml halts Go-side gate sites with no way forward and no spec row

**Location:** `internal/verifytree/verify.go:238`, `internal/websterengine/cardverify.go:57`, `internal/hubgeom/gateslots.go:21`

**Issue:** `hubgeom.GateSlots` wraps a `LoadConfig` failure as `load <path>: <cause>`, and `Verify` returns it as `verifytree: wait for gate slot: read gate limits: load <path>: gate config key "slots": 0; want at least 1`.
That error is a returned error at the webster verify gate, `lyx webster verify`, the round gate, Publish and Finalize, and a `could not acquire a gate slot: ...` line at the card-verify rerun.
Only the absent-file case carries a way forward (`lyx fabric reconcile`, from `ErrConfigAbsent`).
A file that parses but holds a value below 1 is writable through `lyx config gate`, because the hub-wide write validates the file with the template-shaped `configengine.Load` only, so the state is reachable.
`PATTERN-refusal-way-forward` asks every refusal in webster, shed and loom to name its way forward and have a row in `refusal-spec.md`, and the `gate` section of that spec holds the rows for `lyx gate test` only.
`lyx gate test` names `lyx config gate` for the same condition; the Go-side sites name nothing, and none of the new spec rows covers them.

**Fix:** Make the invalid-value error name its way forward in `decodeConfig` (fix the key with `lyx config gate` from the prime), so every site that wraps it inherits the clause, and add one row to the webster and loom sections (or to the grouped raw I/O rows, reworded) for a gate slot that cannot be acquired because `gate.yaml` is absent or invalid.
A test already reaches the card-verify path (`TestRerunCardVerifies_Slotted`, "a failed acquire is a failing card line"); extend its expected text rather than adding a test.
