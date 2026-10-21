# Card 14 — verifytree-unslotted

**Edit:**
- `internal/verifytree#Verify` -> `func Verify(ctx context.Context, p Paths, site Site, command string) (Result, error)`
- `internal/verifytree#Result`
- `internal/verifytree#mustVerify`
- `internal/verifytree#TestVerify_PerCommandRecord`
- `internal/verifytree#TestVerify_Scenario`
- `internal/verifytree#TestVerify_DropsStrandName`
- `internal/verifytree/doc.go`
- `internal/webstercli#websterCLI.runVerify`
- `internal/websterengine#NewVerifyGate`
- `internal/websterengine#newVerifyGate`
- `internal/websterengine#TestVerifyGate_HungVerifyFailsAtOnceWithoutRerun`
- `internal/loomshed#NewVerifyGate` -> `func NewVerifyGate(worktreeRoot, verifyDir, siteLabel string, command, mergeBase func() (string, error), treeScope func() (string, bool, error), round func() (RoundGate, error)) shuttleengine.Gate`
- `internal/loomshed#verifyFailureFindings`
- `internal/loomshed#TestVerifyGate_Scenario`
- `internal/loomshed#roundGateFixture.runGate`
- `internal/loomshed#roundGateFixture.diffGate`
- `internal/loomshed#TestVerifyFailureFindings`
- `internal/landingshed#newVerifyGate`
- `internal/landingshed#verifyGate.verdict`
- `internal/landingshed#Deps`
- `internal/loomcli#landingDeps`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#TestWire_DefaultConfig`
- `internal/shedrecipe#Env`
- `internal/testkit/envkit#nilLegal`
- `internal/landingshed#TestVerifyGate_CheckMapsVerifyResult`
- `internal/shedrecipe#resolvePlainGateClosure`
- `internal/webstercli#TestGateParity_VerifyGate`
- `pattern/PATTERN-verified-tree.md`

**Delete:**
- `internal/verifytree#Timeout`
- `internal/verifytree#TestVerify_SlotGate`

**Intent:**
`verifytree.Verify` stops taking a gate slot and stops applying its 60-minute timeout (D8): every command it runs from here on is one `lyx gate test --wait` call, or such calls joined by `&&`, which take their own slots and are bounded by the gate's pre-tier ceiling and tier timeouts.
That holds from this card on because every site already renders its gate call: the round and darn writer gates since card 10, webster's plan-level gate and `lyx webster verify` since card 12, and Merge-In, Finalize, Publish and darn's tree-scoped command since card 13.

`Verify(ctx, p, site, command)` keeps the dirty check, the skip on a recorded pass, the per-run log and its pruning, the record write after a pass and the strand-name and prebuilt-variable stripping; it drops the pool and timeout parameters, the slot acquire, the waiting marker state it wrote while acquiring, and `Timeout`.
A caller's own cancel stays a returned error with no record.
`Result.TimedOut` now means the failed command's log holds `gateslot.TierTimeoutLine`, found through `gateslot.FindTierTimeout`, with `Detail` naming the tier and its timeout; the status is `StatusFailed` with the command's own exit code.
The marker is written `running` from the start; `MarkerStateWaiting` stays readable for a marker an older binary wrote, and loom status already shows the gate's own wait record ahead of the verify marker, so a waiting gate shows as waiting.

Each caller drops the pool and timeout arguments:
`lyx webster verify`, webster's plan-level gate, loomshed's round gate, whose `NewVerifyGate` loses its `slots` parameter (its recipe caller and tests follow), and landingshed's gate, whose `Deps.GateSlots` goes with loomcli's `landingDeps` line that sets it and envkit's `nilLegal` entry `Landing.GateSlots`.
`shedrecipe.Env.GateSlots` loses its one reader with the recipe caller's argument, so it goes too, with the line in `loomCLI.wire` that sets it from `hubgeom.GateSlots(location)`, `TestWire_DefaultConfig`'s assertion of it and envkit `nilLegal`'s `GateSlots` entry.
Each message that named `verifytree.Timeout` names `Result.Detail` instead: webster's timed-out report, loomshed's failure findings and landingshed's Stuck reason.
The Stuck reason also names, beside fixing a hanging test, the step for a tier that is slow but healthy: rewriting the tier whole from the prime with `lyx config gate --set 'tiers.tier<N>={...}'`, carrying every key of the tier and the raised `timeout_min` (D1), since a dotted key under the tier would write an entry the loader refuses.

`doc.go` drops the slot and timeout paragraphs and states the timeout reading.
`pattern/PATTERN-verified-tree.md` drops its slot line; card 13 already named every site's rendered gate call there.

Tests: `TestVerify_SlotGate` goes with the slot.
`TestVerify_Scenario`'s timeout row becomes a command that prints the tier timeout line and exits 1, read back as `TimedOut` with the tier in `Detail`.
`TestVerifyGate_HungVerifyFailsAtOnceWithoutRerun` and `TestVerifyFailureFindings` assert the detail-based wording, and `TestVerifyGate_CheckMapsVerifyResult` asserts it with the whole-tier `--set` step.
`TestVerify_DropsStrandName` drops its slotted row, which has no pool left to pass.
The remaining tests change only their call shape.

**ImpactSummary:** one exported function re-signed with its four production sites and their tests; one exported constant, one loomshed parameter and the recipe env's slot pool removed; no verify record or log format changes.
