---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/plan/09-webster-spawn-engine-checks.md:22
    summary: Card 9 says a missing role entry keeps today's refusal, but Run has no such refusal for the recovery role and its Master refusal fires after the state mutation the new gate precedes
    fix: In card 9's Intent, replace the sentence "a missing role entry keeps today's refusal" with "a role with no entry in `deps.Roles` is skipped by the gate, so Master's `no resolved model-spec` refusal in `Run` and the recovery one in `recoverSpawn` fire where they do today".
---

Every focus item was traced against the tree; the first three come out clean and the fourth leaves the one finding below.

- **Card 8's rename of `RunDeps.Engine`.**
  Each reader of the old field is a target of card 8 or an earlier card: `websterEntry`, `zeroEnvField`, `simpleEntryCases`, `TestWebsterEntry_SeamFields`, battencli's `intentionallyNil`, `shedfake`'s `WebsterSeams` and its non-nil test, `loomCLI.wire` and `TestWire_DefaultConfig`.
  So are the websterengine readers (`uncommittedPathsBlock`, `RecoverSpawnOrAttach`, `awaitTerminal`, `ownTrackedPaths`, `RecordBatch`, `auditTerminalFork`, `Run` and its pending-findings helpers) and every test literal that sets `Engine:` on a deps value.
  `envkit`'s reflection walk needs no change, because `Engines` is filled by `WebsterSeams` and `loomcli`'s `intentionallyNil` never listed the field.
- **Card 10's `RunDeps.InstructionFiles`.**
  `nilLegal` keeps `NilSeams(FullEnv)` empty and battencli's list unchanged, and `loomCLI.wire` fills the field, so loomcli's list needs no entry.
  Both `RunDeps` constructors (`loomCLI.wire`, `websterCLI.runDeps`) are targets of card 10, and a test asserts each wiring sets the field.
- **Cards 10 and 11 refusals.**
  Every step they name is one the caller can take: `lyx config webster` and `lyx config loom` write per-worktree keys that `hubWriteRefusal` does not guard, and `shuttle.yaml` is not hub-wide.
  The rows' triggers are reachable past card 9's gate.
  `await-batch` and `validate` reach `executionBatches` with a nil state, `rebaseline` calls `MerriamBase` directly, and an engine that audits forks but lacks `background-wake` passes the gate and meets Master's `Start` refusal.
  `lyx orch start` runs on the default engine with no alias, so the generic clause's `engine:` half is the step open to it; `lyx burler run` takes the `DefaultEngine` form.
- **Index, cross-references and `## verify:`.**
  The Card Index, the card headings, the `**Commit:**` lines and every card number cited in another card's prose agree, and the verify section follows the template's order.
  The baseline `lyx loom validate-plan --overlay` run with `{}` is ok, with only an informational `create-new-unit` note for the PATTERN background file.

### [LOW] Card 9's "missing role entry keeps today's refusal" is not true for the new gate

**Location:** `_lyx/plan/09-webster-spawn-engine-checks.md:22`

**Issue:** The gate sits before `AcquireStateMutation`, but `Run`'s only missing-role refusal, `no resolved model-spec for role "master"`, is at `internal/websterengine/runlevel.go:683`, after the state lock, `LoadState` and the reclaim.
`Run` has no recovery-role refusal at all; that one lives only in `recoverSpawn` (`internal/websterengine/recoverbatch.go:300`).
A fixer reading the sentence literally can add a missing-role refusal to the gate for the recovery role, or reorder Master's, which the decision record does not license, and it would need a refusal row.
Skipping a role with no entry in the gate is what the sentence must mean, and the card should say so.
