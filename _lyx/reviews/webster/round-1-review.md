---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/frictionengine/spec.go:34
    summary: buildReflectionSpec doc comment still says the spec takes Model/Effort/Version off the resolved value, but the code now builds the choice with shuttleengine.NewModelChoice
    fix: In the doc comment of buildReflectionSpec in internal/frictionengine/spec.go, replace "then Model/Effort/Version off the resolved value" with "then shuttleengine.NewModelChoice(resolved)", keeping one sentence per line.
  - id: F2
    severity: LOW
    class: consistency
    location: internal/mergeresolve/spec.go:40
    summary: buildConflictSpec doc comment still says the spec takes Model/Effort/Version off the resolved value, but the code now builds the choice with shuttleengine.NewModelChoice
    fix: In the doc comment of buildConflictSpec in internal/mergeresolve/spec.go, replace "then Model/Effort/Version off the resolved value" with "then shuttleengine.NewModelChoice(resolved)", keeping one sentence per line.
---

Range: `git diff $(git merge-base main HEAD)..HEAD`.
`_lyx/loom/rework/` does not exist, so step 4 of the rubric's range derivation applies, and `product.parent` is `main`.
`lyx gate test ./...`, `lyx loom lint-pattern` and `lyx loom lint-comments` are all green on the reviewed tree.

### [LOW] buildReflectionSpec comment describes the retired field copy

**Location:** internal/frictionengine/spec.go:34
**Issue:** Card 1 moved the spec's model fields onto `ModelChoice: shuttleengine.NewModelChoice(resolved)`.
The comment above `buildReflectionSpec` still describes the old three-field read, so it contradicts the code below it.

### [LOW] buildConflictSpec comment describes the retired field copy

**Location:** internal/mergeresolve/spec.go:40
**Issue:** The same stale sentence as F1, in the comment above `buildConflictSpec`, whose body now calls `shuttleengine.NewModelChoice(resolved)`.

## Focus checks

- Cards 1-5: every resolver that reads a `modelspec.Resolved` builds its choice through `NewModelChoice`.
  The composite literals left in production (`burlercli/run.go`, `orchcli/start.go`, `shuttlecli/run.go`) take a provider model string from a flag or `orch.yaml`, not a resolved alias, which cards 1 and 3 keep as `Engine`-empty literals.
  Timeouts and spawn budgets stay beside the choice in `ArbiterSettings`, `AlignmentSettings`, `TacklerSettings` and treadle's `Profile`.
  The Delete groups hold: `burlerengine.ModelChoice`, `DriverSettings`, `AuditEngineFrom` and `auditengine_test.go` have no remaining reference.
- Cards 7-9: `State.MasterEngine`, `BatchState.Engine` and `RecoveryEngines` are written at Master spawn, begin-batch and recovery spawn, and each reader goes through `sessionAuditEngine`.
  `Run`'s role check sits after the plan-approved and `ErrNilBatcher` checks and before `AcquireStateMutation`.
  `RecoverSpawnOrAttach`'s check sits after the attach return and ahead of the exhausted-recoveries refusal, the fabric-reference acceptance, `archiveStaleReport` and `StopStrand`.
- Cards 11-13: `ModelKey.Wrap` is applied at the seat start, Master's `Run`, and the driver arm.
  Every new refusal has its `refusal-spec.md` row (session engine not registered, engine-less session, master engine not registered, master and driver requirement rows, the four loom seat rows, the MultiLLM seat row, and role engine without fork audit).
- Card 14: `TestModelChoiceMappingRule` covers each way the scan can see a `Resolved`, including the `NewModelChoice` exemption and its same-file counterexample.
  The `PATTERN.md` entry lands in the same commit, and the five trimmed entries lose only clauses their background files already carry (`lint-pattern` passes).
- Test economy: the diff adds two top-level tests, `TestOpenCmd_ModelAliasEngine` and `TestModelChoiceMappingRule`.
  `TestOpenCmd_ModelAliasEngine` pins `open --model`, which no existing shuttlecli test covers.
  The other new coverage is rows in existing tests, so no duplicate and no uncovered added behavior turned up.
