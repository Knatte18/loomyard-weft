# Card 10 — MerriamBase counts the instruction files Master's engine names

**Create:**
- `plan:internal/websterengine#MasterInstructionFiles` -> `func MasterInstructionFiles(engines AuditEngines, master shuttleengine.ModelChoice) func() ([]string, error)`

**Edit:**
- `internal/websterengine#MerriamBase` -> `func MerriamBase(geom Geometry, instructionFiles []string) (batcher.StartBase, error)`
- `internal/websterengine#RunDeps`
- `internal/websterengine#Run`
- `internal/websterengine#autoRebaseline`
- `internal/websterengine/doc.go`
- `internal/webstercli#websterCLI`
- `internal/webstercli#websterCLI.wireHub`
- `internal/webstercli#websterCLI.wireStandalone`
- `internal/webstercli#websterCLI.executionBatches`
- `internal/webstercli#websterCLI.rebaselineCmd`
- `internal/webstercli#websterCLI.runDeps`
- `internal/loomcli#loomCLI.wire`
- `internal/testkit/envkit#nilLegal`
- `internal/webstercli#TestWire_StandaloneMode`
- `internal/webstercli#TestWire_ModeHubSelectsHubMode`
- `internal/websterengine_test#TestMerriamBase`
- `internal/webstercli#TestValidateCmd_BatchesFlag`
- `internal/webstercli#TestValidateCmd_ScopeFollowsRunProgress`
- `internal/webstercli#TestRebaselineCmd_Refusals`
- `internal/webstercli#TestRecordBatchCmd_Envelope`
- `internal/webstercli#TestResetCmd`
- `internal/webstercli#TestRecoverBatchCmd_RunningThenTerminal`
- `internal/loomcli#TestWire_DefaultConfig`
- `contracts/specs/refusal-spec.md`

**Intent:**
D7.
`MerriamBase` counts the files whose names it is told, each at the worktree root when present, in place of naming `CLAUDE.md` and `CLAUDE.local.md`; an absent file adds nothing and any other read failure is still refused through `merriamBaseError`.
`MasterInstructionFiles` returns a lazy lister: when called it looks up Master's choice engine in the told set, returns the names of its `shuttleengine.InstructionFileLister` capability, none for an engine without it, and for an engine the set does not hold the lookup's error wrapped with the way forward `set master in webster.yaml to an alias whose engine the set holds with lyx config webster, then re-run the verb`.
`websterengine` builds no engine set; it applies the lookup it is told.
`RunDeps` gains `InstructionFiles func() ([]string, error)` (nil meaning none), which `Run`'s partition formation and `autoRebaseline` call right before their `MerriamBase` call, returning its error unchanged.
`Run` still calls the lister before each `MerriamBase` call, but card 9's `roleAuditEngine` entry gate has already refused a `master` engine the set does not hold, so `Run` propagates the lister's error only as an internal error.
`websterCLI` gains the same lister, nil likewise meaning none, so a test literal that sets none keeps counting no instruction file, set by both wiring prologues, and `executionBatches` (no state) and `rebaselineCmd` call it before `MerriamBase`; `loomCLI.wire` fills `RunDeps.InstructionFiles`, and `websterCLI.runDeps` copies the CLI's lister into it, so the `run` verb counts the files Master's engine names.
`executionBatches` and `rebaselineCmd` return the lister's error without the `way forward: transient, re-run the verb` clause they append to a `MerriamBase` failure, so the refusal carries only the `master` way forward.
A nil `RunDeps.InstructionFiles` means none, so `envkit`'s `nilLegal` gains `WebsterDeps.InstructionFiles`, which keeps `NilSeams(FullEnv)` empty and leaves battencli's `intentionallyNil` list unchanged.
The lister runs only where a fresh partition is formed or a rebaseline prices the tail, so on a run with a state `record-batch`, `reset`, `validate` without `--batches`, a `recover-batch` attach or await, and loom `status` and `pause` never call it and are never refused by the `master` key.
`doc.go`'s start-base paragraph says the files are the ones Master's engine names.

`contracts/specs/refusal-spec.md`'s webster section gains the row `master engine not registered`: `validate` or `await-batch` run with no state, `validate --batches` or `rebaseline` finds `master`'s engine missing from the engine set, correctness halt, with the way forward above;
`run` is refused earlier by the `role engine without fork audit` row.
Its existing `Merriam start base unreadable` row is reworded from a present `CLAUDE.md` or `CLAUDE.local.md` to a present instruction file Master's engine names.

Tests:
- `TestMerriamBase` passes the names: `CLAUDE.md` and `CLAUDE.local.md` grow the base by their lines as today; a list of `AGENTS.md` alone counts that file and ignores a present `CLAUDE.md`; an empty list adds nothing over a present `CLAUDE.md`; a named file that is a directory is refused naming it.
  Its `MasterInstructionFiles` subtests, named for that function, run it over a set holding a fake with `InstructionFiles` (`AGENTS.md`), a plain fake (none) and a master choice naming an engine the set lacks (refused naming `master` and `ErrEngineNotRegistered` wrapped).
- The refusal at each webstercli `MerriamBase` caller, with a lister that fails as for an unregistered engine: `TestValidateCmd_BatchesFlag` gains a row where `validate --batches` refuses naming `master`; `TestRebaselineCmd_Refusals` gains a row where `rebaseline` refuses naming `master`; each row asserts the message holds the `set master in webster.yaml` way forward and no `transient`.
- `TestWire_StandaloneMode` and `TestWire_ModeHubSelectsHubMode` assert `runDeps().InstructionFiles` is non-nil after wiring.
- The verbs that must still run on a run with a state, with the same failing lister on the `websterCLI`: `TestValidateCmd_ScopeFollowsRunProgress` (`validate` without `--batches`), `TestRecordBatchCmd_Envelope` (`record-batch`), `TestResetCmd` (`reset`) and `TestRecoverBatchCmd_RunningThenTerminal` (the second, attaching call) each gain a row that succeeds as before.
- `TestWire_DefaultConfig` asserts `runDeps.InstructionFiles` is set and that `wire` resolved no engine for it, so loom `status` and `pause`, which run `wire`, are never refused by the `master` key.

**ImpactSummary:** `MerriamBase` takes its file names; its four callers and the three wirings change, and the engine lookup they need is lazy so no spawn-free verb on a run with a state can be refused by it.

**Commit:** 10: MerriamBase counts the instruction files Master's engine names
