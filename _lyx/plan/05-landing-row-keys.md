# Card 5 — landing-row-keys

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#verifyGate`
- `internal/landingshed#newVerifyGate`
- `internal/landingshed#verifyGate.check`
- `internal/landingshed#verifyGate.recordFailure`
- `internal/shedrecipe#mergeInEntry`
- `internal/shedrecipe#finalizeEntry`
- `internal/loomcli#verifySource`
- `internal/loomcli#planVerifySource`
- `internal/loomcli#darnVerifySource`
- `internal/loomcli#landingDeps`
- `internal/testkit/envkit#fixedGateTiers`
- `internal/landingshed#TestVerifyGate_CheckMapsVerifyResult`
- `internal/landingshed#TestMergeIn_PlanVerifyFailureRecord`
- `internal/landingshed#TestFinalizeVerify_Fail`
- `internal/loomcli#TestLandingDeps_VerifyCommandReadsPlanAtCallTime`
- `internal/verifytree#FailureKindLanding`
- `internal/landingshed#newGateScratch`
- `internal/landingshed#TestVerifyGate_RealVerifytreeScenario`
- `internal/landingshed#TestMergeIn_Fixer`
- `internal/landingshed#newMergeInFixture`
- `internal/landingshed#newTestDeps`
- `internal/landingshed#newFixerRunFixture`
- `internal/landingshed#newGateFixture` -> `func newGateFixture(t *testing.T, gateKey, command string, closureErr error) *gateFixture`
- `internal/landingshed#newFinalizeGateFixture`
- `internal/landingshed#newFinalizeVerifyFixture`
- `internal/landingshed#TestFinalize_MergeJudgement`
- `internal/landingshed#TestVerifyGate_WaitMark`
- `internal/landingshed#TestVerifyGate_CheckWithoutCommandSkipsVerify`
- `internal/landingshed#TestVerifyGate_ClosureError`
- `internal/landingshed#TestVerifyGate_Cancellation`
- `internal/landingshed#TestVerifyGate_Clean`
- `internal/loomcli#TestDarnTreeScope_ClassifiesAtEachCall`
- `internal/testkit/envkit#LandingDeps`
- `internal/shedrecipe#TestSimpleEntries_HappyPath`
- `internal/landingshed/doc.go`
- `internal/verifytree/doc.go`

**Intent:**
Merge-In and Finalize each run their own gate key (D1), and the merge fixer runs the key of the row that spawned it.

`mergeInEntry` and `finalizeEntry` accept one config key, `gate_key`, a string, and refuse any other key as today; the row's key is that value, else its row name in lower case, `merge-in` and `finalize`.
A non-string `gate_key` is refused naming the row, the key and the way forward of setting it to the name of a `gate.yaml` key, a string, or removing it.
Each entry copies `env.Landing` and sets the copy's new `Deps.GateKey` field to the key before it calls `landingshed.NewMergeIn` or `NewFinalize`, so Env stays run-wide; the producers read `GateKey` as set, with no default for an empty one.
Bound: `gate_key` selects only which key of the hub's `gate.yaml` the row runs.

`Deps.VerifyCommand` becomes `func(gateKey string) (string, error)`, read with the row's key at each gate call; `verifyGate` keeps the key in a new field that `newVerifyGate` fills.
The merge fixer already judges each turn through the spawning row's `verifyGate`, so it runs that row's key with no change of its own.
`verifyGate.recordFailure` writes the row's key as the record's kind and reads that key's tiers through `Deps.GateTiers` for a log that names no failing tier, replacing card 1's interim `finalize` lookup.
`verifytree`'s `FailureKindLanding` keeps naming the retired `landing` kind, and its doc and the package doc say a record carrying it reads as the key of the row that reads it, Merge-In, Finalize or the merge fixer they spawn, since the record names no row and only those rows read it.
No production code reads `LandingFailure.Kind` today (Merge-In, Finalize and the fixer read only its log path), so this reading is the record's documented meaning and adds no branch.

`Deps.BaseCommand` is the fix rows' `code-burler-fix` call in loom and darn alike, so a landing pass keeps the entry the Code-Burler-Fix lint base reads.
`verifySource.command` takes the key; `planVerifySource` reads it through card 4's `gateCallSource`, and `darnVerifySource` answers the tree-scoped command for a tree-scoped diff and the key's call otherwise.
`envkit`'s `fixedGateTiers` answers the eight keys.

`TestMergeIn_PlanVerifyFailureRecord` asserts the record's kind is `merge-in`, and `TestFinalizeVerify_Fail` asserts `finalize`.
`TestSimpleEntries_HappyPath` gains rows building `MergeIn` and `Finalize` over an env whose `Landing.VerifyCommand` records the key it is called with: with no config the gates read `merge-in` and `finalize`, with `gate_key: custom` they read `custom`, and `gate_key: 3` is refused with the way forward.
`TestMergeIn_Fixer` asserts the fixer's turn gate calls `VerifyCommand` with the spawning row's key.
`TestVerifyGate_CheckMapsVerifyResult` and `TestLandingDeps_VerifyCommandReadsPlanAtCallTime` pass the key.
The callers of the re-signed `Deps.VerifyCommand`, found by a search for `VerifyCommand` over the landing tests and envkit as well, move onto it: landingshed's `newGateScratch` and the `integration`-tagged `TestVerifyGate_RealVerifytreeScenario`, which sets `deps.VerifyCommand` itself, set a keyed closure, and the hand-built Deps and gates, found by a search for `Deps{` and `verifyGate{` over the landing tests, set the row key each builds for:
the Deps of `envkit.LandingDeps`, `newGateScratch`, `newMergeInFixture` and `newFixerRunFixture` set `GateKey` to `merge-in`, and `newTestDeps`, which the Finalize fixtures build on, to `finalize`;
`newGateFixture`, whose `verifyGate` literal the Merge-In, fixer, Finalize and `verifyGate` tests use without `newVerifyGate`, takes the row key as a parameter and sets the gate's key field from it, the Merge-In and fixer fixtures passing `merge-in`, the Finalize fixtures and `TestFinalize_MergeJudgement` `finalize`, and the `TestVerifyGate_` tests either; and `TestDarnTreeScope_ClassifiesAtEachCall` calls `deps.VerifyCommand("finalize")`.
The landingshed doc's verify paragraphs name each row's key where they name the `landing` gate.

**ImpactSummary:** Keys landing's verify command by row and adds `Deps.GateKey`; touches the two landing entries, the landing verify gate, loom's landing wiring and envkit.
