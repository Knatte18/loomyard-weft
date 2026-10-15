# Lens review

Every lens fork returned a report.

## criteria-coverage

- CC1, MEDIUM, D8's test method: the plan injected canned `ErrRequirementsUnmet` errors and no row ran shuttle's real `Start` check, so the empty-key-against-default case went untested. Fixed as proposed: card 11's `TestEngine_Run_StartFailureEndingTheRunStopsEveryStartedSeat` rows and burlercli's `TestRunCmd` row, plus card 12's `TestRun_RefusesBeforeSpawn` (new `realMasterStarter`) and `TestStartLLMDriverArm` (`runnerDriverStarter`) rows, now run a real `Runner` over `shuttlefake.EngineSet` with empty-engine choices and assert the key, engine `fake` and no `AddStrand`. Merged with BC2.
- CC2, MEDIUM, D7: the still-runs verbs and the other `MerriamBase` refusal callers had no rows. Fixed as proposed on card 10: refusal rows for `run`'s partition formation (no state and `--fresh`), auto-rebaseline, `validate --batches` and `rebaseline`; still-runs rows for `validate`, `record-batch`, `reset` and a `recover-batch` attach; and a `TestWire_DefaultConfig` assertion for loom `status`/`pause`. Merged with BC7 and RR2.
- CC3, MEDIUM, escape form and spec engines: merge-resolve and the Bouncer specs had no engine assertion, and most resolvers had no escape-form row. Fixed differently. Proposed: an escape-form row per resolver. Not made, because each resolver goes through `Registry.Resolve` then `NewModelChoice`, which card 13's scan enforces, so one end-to-end row proves the escape form for all of them (also TE2). Instead: card 1 adds that row to `TestNewModelChoice` and a `TestBuildConflictSpec_SkillsAndParentDirective` engine assertion, and card 3 adds spawned-spec `Engine` assertions to `TestBouncer_JudgeCall_Verdicts` and `TestBouncer_SeedCall_HappyPath`.
- CC4, MEDIUM, D5: merge-resolve and treadle's targeting spec had no "no `Requires`" assertion. Fixed as proposed: `TestBuildConflictSpec_SkillsAndParentDirective` (card 1), and `TestSpawnedRoles_SkillAndParentDirective` covering the circling, milestone and targeting specs (card 5). The attempt spec is built by a `RoundRunner` that no production code implements.
- CC5, LOW, D4: no row covered an absent record engine. Fixed as proposed: card 1 gains an attach row over a `run.json` with no `engine` key, asserting `Result.Engine` `"a"`.
- CC6, LOW: the refused `recover-batch` row did not assert `state.json` unchanged. Fixed as proposed on card 9. Merged with BC5.
- CC7, LOW, D8: a MultiLLM seat's zero `ModelKey` left its refusal with no way forward. Fixed differently. Proposed: give `multiLLMSeat` a key. Not made, because a recipe seat's model comes from the recipe row, not a `lyx config` key. Instead: card 11's `Wrap` writes a seat way forward for the zero key, its shed section gains a row, and `TestRunner_Requires` has a zero-key row. Merged with BC1.
- CC8, LOW, docs constraint: `alignment` and `sheddriver` had no `doc.go` listed. Fixed as proposed on card 4. `loomengine` has no `doc.go`, and `darnengine`'s doc does not describe the seat's model fields.

## branch-coverage

- BC1, MEDIUM: a zero-key seat refusal had no way forward. Fixed differently, see CC7.
- BC2, MEDIUM: the loom rows had no test through a fake engine set. Fixed as proposed, see CC1.
- BC3, MEDIUM: `RecordBatch` and `auditTerminalFork` had no lookup-refusal rows. Fixed as proposed: card 8's `TestRecordBatch_AuditOutcomes` gains rows for an unregistered engine (refused, nothing archived), an engine-less batch on a default without audit, and a terminal fork batch routed through its recorded engine.
- BC4, MEDIUM: `awaitTerminal`'s refusal branches were unreached. Fixed as proposed: card 8's `TestRecoverBatch_SecondCall` gains rows for an unregistered engine, an absent key on a default without audit, and an empty `RecoverySessions`.
- BC5, MEDIUM: `recover-batch`'s refusal did not assert `state.json` unchanged and had no unregistered-engine row. Fixed as proposed on card 9.
- BC6, MEDIUM: no row refused an empty choice resolving to a default without fork audit. Fixed as proposed: a card 9 `TestRun_RefusesBeforeSpawn` row.
- BC7, MEDIUM: D7's refusal paths and the nil lister were unrowed. Fixed as proposed, see CC2. The nil-`InstructionFiles` path is every existing `Run` test, whose fixture leaves it nil.
- BC8, LOW: the held-engine-without-audit internal error and the nil-`Engines` fallback had no rows. Fixed as proposed: card 8 `TestLoadRunWrites` rows. Merged with RR4.
- BC9, LOW: the first-sighting rule had no row. Fixed as proposed: a card 8 `TestLoadRunWrites` row.
- BC10, LOW: the burler fix-half wrap and pass-through had no rows. Fixed differently. Proposed: add rows for the fix half. Not made, because card 11 no longer wraps in `runCmd`, and the fixer is a lone, non-forking seat that requires nothing. Instead `runCmd` sets `ReviewKey` to `ModelKey{DefaultEngine: true}`, and the pass-through of a non-requirements error is a `TestRunner_Requires` `Wrap` row.
- BC11, LOW: the scan's `var` and range branches had no rows. Fixed as proposed on card 13.
- BC12, LOW: no wiring test showed wiring over a default without fork audit now succeeds. Disputed, every fix worse: `webstercli`'s `wireHub`/`wireStandalone` and `loomCLI.wire` build the set through `shuttleengines.New(shuttleCfg)`, whose registry holds only `claude`, which audits forks (commit 88ea645bb: `New` refuses an engine that declares forks without audit). So no config can produce that default, and a test would need a production seam added only to inject one. `AuditEngineFrom`, the only code that refused, is deleted on card 8, and the acceptance case is pinned at `Run` (card 9's `TestRun_MasterSpawn` row).

## row-reachability

- RR1, MEDIUM: the `masterEngine` assertion could not fail, because `seedShuttleRunState` writes no engine (`internal/websterengine/runlevel_test.go:154-159`). Fixed as proposed: card 7 lists `seedShuttleRunState`, which now writes `shuttlefake.EngineName`, and `TestRun_MasterSpawn` asserts `masterEngine` `fake`.
- RR2, MEDIUM: only `validate` had a still-runs row. Fixed as proposed, see CC2.
- RR3, MEDIUM: the burler row did not run the cluster-fan path. Fixed as proposed: card 11's `TestRunCmd` row uses a real `Runner` and a cluster-fan profile, and asserts no `AddStrand`.
- RR4, LOW: the held-engine-without-audit branch had no row. Fixed as proposed, see BC8.
- RR5, LOW: the removed-run-directory row could not tell right from wrong. Fixed differently. Proposed: reach `reset` and `recover-batch`. Done for `reset` only: the row moves to card 8's `TestPlanReset`, a recovery engine recorded with no run directory. `recover-batch`'s write-history read is the same `loadRunWrites` over the state, which the moved row and the `TestLoadRunWrites` rows reach.

## forced-edits

- FE1, MEDIUM: `websterCLI`'s lister had no nil meaning, so existing test literals would panic. Fixed as proposed: card 10 says a nil `websterCLI` lister means none.
- FE2, LOW: `auditengine.go`'s header goes stale. Fixed differently. Proposed: list the file. Not made, because card 8 lists `AuditEngine` in that file and `redundant-file-target` refuses the file beside it. Instead card 8's Intent prescribes the header rewrite.
- FE3, LOW: `roles.go`'s header goes stale. Fixed differently: card 7's Intent prescribes the rewrite, for the same `redundant-file-target` reason (`ResolveRoles` is listed).
- FE4, LOW: `models.go`'s header goes stale. Fixed differently: card 3's Intent prescribes the rewrite (`RoundModels` is listed).

## test-economy

- TE1, LOW: `TestOpenCmd_ModelAliasEngine` should be a row of an existing real-`Runner` test. Disputed, every fix worse: the two real-`Runner` tests the finding names are `TestSendCmd_AppendsMessageTail` (`internal/shuttlecli/cli_test.go:508`) and `TestInterruptAndSendCmd_FailWithTheGuardsMessage` (`:597`), which pin other verbs' behavior. `TestRunCLI_Run_FlagValidation` runs on a deliberately nil runner (`cli_test.go:96`). No existing test covers `open` against a runner.
- TE2, LOW: the per-resolver escape-form rows repeat `TestNewModelChoice`'s behavior. Fixed as proposed: cards 2, 3 and 7 drop them, and card 1's end-to-end row replaces them (see CC3).
- TE3, LOW: the `MasterInstructionFiles` rows ran under `TestMerriamBase`'s name. Fixed as proposed: card 10 names those subtests for `MasterInstructionFiles`.
- TE4, LOW: the `Wrap` rows would build the requires rig. Fixed as proposed: card 11 makes them plain table rows that build no rig.
