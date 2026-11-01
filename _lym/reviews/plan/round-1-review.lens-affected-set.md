```yaml
lens: affected-set
outcome: reported
```

Task: affected-set lens. For each change a card makes, find what reads or asserts it, and which cards cover that set.

**Per-change sets and covering cards**

- **Card 1 (`keep_warm_max_reruns` key).** Readers: `TestLoadConfig_TemplateDefaultsResolve`, the new test, the `//lym:seam` Uses tests and the doc. Covered by card 1.
- **Card 2 (`Engine.KeepWarmCaps` and the claude.yaml caps).** Readers: the three implementers (claude `command.go:170`, `shuttlefake.go:259`, `fakes_test.go:461`; no other implementer exists), `TestLoadConfig`, `TestPrepare_RejectedBeforeArtifacts`, `ttlConfig`, the registry `CheckLoad` and both docs. Covered by card 2.
- **Card 3 (`shuttleengines.KeepWarm`).** Readers: `allowedImports` and the doc. Covered by card 3.
- **Card 4 (await `--cap` default, 300000 → 3600000 ms).** Readers: the `await.go` help, the `DefaultAwaitCap` doc, `shuttleengine/doc.go:136` and the shuttle paragraph at `docs/overview.md:419`. `TestAwaitVerb` needs no change. Other 300000 hits (such as `configsync.go:49`'s token threshold) are unrelated. Covered by card 4.
- **Card 5 (strand removal).** Readers: `runDeploy`, `runRowCommand`, `cardverify.runLine` and the cardverify doc. Covered by card 5.
- **Card 6 (`verifyrun.RunTied`).** Readers: `run`, `TestMain` and the package comment. Covered by card 6.
- **Card 7 (cardverify takes a context).** Readers: `runCardVerify` (`gatecli/cardverify.go:102`), `rerunCardVerifies` (`websterengine/cardverify.go:17`, whose caller `recordbatch.go:534` keeps its signature), the `TestRunner` calls at `runner_test.go:37/52/55/118/162` and the doc. Covered by card 7.
- **Card 8 (`RoleGate`).** Reader: the runproc doc. Covered by card 8.
- **Card 9 (record fields, report split).** Readers: `runReport`, `reportCmd`, `TestGateReport`, `TestWriteRunRecord`, `TestGateTest_Scenario` and the docs. No other reader of the report's row keys exists. Covered by card 9.
- **Card 11 (in-process calls wait uncapped; only a capped front answers 75; help text drops the slot-wait cap).** Readers:
  - help text at `cli.go:76-85` (card 11)
  - `cli_test.go:59-66`, the help subtest of `TestGateTest_RefusesBeforeResolvingTheHub` (no card)
  - `gatecli/doc.go:157-158,337` (card 11)
  - `run.go` `haltExitCode` and `answerSlotBusy` (card 11)
  - `rungate.go` (card 11)
  - scenario rows at `test_integration_test.go:899,1008,1038` (card 11)
  - `gateslot/doc.go:51`, `docs/overview.md:494` and `docs/dotnet-hub.md:126` (card 11)
  - shellguard `rows.go:161,176` and `rows_test.go:45` (`TestCheckTexts`) (card 15)
  - `webster-body-implementer.md:100-102` (card 15)
  - `loom-plan-spec.md:595` and `loomshed/plancontext.go:147` (no card)
- **Card 11 (`newCommand` signature).** Callers: `Command`, `runGateHelper` (`test_integration_test.go:123`) and card 13. Covered by cards 11 and 13.
- **Card 13 (`cardVerifyCmd` and `runCardVerify` signatures, card-verify help).** Readers: `newCommand`, the `TestGateCardVerify` annotation subtest and the gatecli doc. Covered by card 13.
- **Card 14 (agent gate calls drop `--wait`).** Readers:
  - `TestConfig_TierUse` (`config_test.go:763`)
  - `TestCardGateCommand`, `TestRenderCardGates_…`, `TestRenderBatchGate` and `TestRenameBoundary_…` (`cardgate_test.go:47-148`)
  - `TestBeginBatch_PromptFile` (`beginbatch_test.go:407`) and `TestRecoverSpawnOrAttach` (`recoverbatch_test.go:1692`)
  - `TestFixVerifySpawnOrAttach` (`verifyfixer_test.go:168`) and `TestRebaselineCmd_…` (`webstercli/verbs_test.go:1036`)
  - `websterengine/doc.go:551` and the `PATTERN-test-speed` gate-keys sentence
  - `loom-plan-spec.md:336,344` (card 14)
  - `loom-plan-spec.md:320` (card 14 names only its "card and batch gate sentences")

  `refscanner_test.go:68` is a ref-scanner fixture and is unaffected.
- **Card 15 (gate rule and stencils).** Readers: `Rows`, `refuseRawRow` and `TestCheckTexts`; the implementer, darn and drift stencils. Covered by card 15.

**Findings**

1. **MEDIUM, design, card 11.** Card 11's help-text change misses a member of its affected set. The test verb's help drops "Without --wait it waits for the slots for up to %s", but the help subtest of `TestGateTest_RefusesBeforeResolvingTheHub` asserts `"for up to " + shuttleengine.DefaultAwaitCap.String()`. No card edits it, so card 11's own tier0 gate fails. The fix adds that test to card 11's Edit list and rewrites the subtest's assertion against the new help, or drops it as static prose. Evidence: `internal/gatecli/cli_test.go:59-66`, `internal/gatecli/cli.go:76-85`.
2. **LOW, consistency, card 11 and the plan's coverage of the 75 answer.** Once only a capped front answers 75, the stated reason for the `verify-gate-no-wait` check becomes false. A line without `--wait` no longer "exits 75 on a busy slot": an in-process call waits for its slots with no cap. Both the plan-gate message and the spec sentence still say it, and no card lists either; card 14 edits `loom-plan-spec.md`, but only its card and batch gate sentences. Evidence: `internal/loomshed/plancontext.go:147`, `contracts/specs/loom-plan-spec.md:595`.
3. **LOW, consistency, card 14.** The spec's general gate-keys sentence "every site runs `lym gate test --wait --gate &lt;key&gt;`" becomes false for the agent sites (card, batch, `webster-fix`). Card 14 rewrites the twin sentence in `pattern/PATTERN-test-speed.md`, but names only the spec's "card and batch gate sentences", so this one may be left unchanged. Evidence: `contracts/specs/loom-plan-spec.md:320`; card 14 Intent at `_lym/plan/14-agent-gate-calls.md`.

**Tool calls**

1. Grep for callers of `newCommand`, `SlotBusyExit`, `helperSlotWaitEnv` and `answerSlotBusy`, the `PromptCacheLifetime` implementers, the `RunLine`/`RunCards` callers and the "no gate slot is free" / "exits 75" texts, repo-wide: the manifest held definitions, not every caller or asserter.
2. The test function enclosing `rows_test.go:45`, the `verify-gate-no-wait` sources, the files asserting rendered `--wait --gate card|batch|webster-fix` strings, and readers of `lym gate report` rows: to map asserters to cards.
3. The enclosing test function of each rendered-call assertion: to check card 14's test list against them.
4. The `configsync.go` 300000 hit, the `verify-gate-no-wait` detail message and CLI-index mentions of the gate verbs: to decide whether they belong to the changed sets.
