# Card 14 — agent-gate-calls

**Create:**
- `internal/gateslot#Config.AgentGateCall`

**Edit:**
- `internal/websterengine#cardGateCommand`
- `internal/websterengine#renderBatchGate`
- `internal/websterengine#FixVerifySpawnOrAttach`
- `internal/websterengine#Run`
- `//internal/websterengine/doc.go`
- `internal/gateslot#TestConfig_TierUse`
- `internal/websterengine#TestCardGateCommand`
- `internal/websterengine#TestRenderCardGates_OneBulletPerCardWithAStepSubBulletEach`
- `internal/websterengine#TestRenderBatchGate`
- `internal/websterengine#TestRenameBoundary_BatchesCheckAndGateTheirOwnTree`
- `internal/websterengine_test#TestBeginBatch_PromptFile`
- `internal/websterengine_test#TestRecoverSpawnOrAttach`
- `internal/websterengine_test#TestFixVerifySpawnOrAttach`
- `internal/webstercli#TestRebaselineCmd_EditedCardOfFailedBatchThenRecover`
- `//contracts/specs/loom-plan-spec.md`
- `//pattern/PATTERN-test-speed.md`
- `internal/loomshed#verifyLineFindings`

**Extends:**
- `//internal/gateslot/doc.go`

**Uses:**
- `internal/gateslot#Config.GateCall`
- `//internal/loomshed/doc.go`

**Intent:** D7: a gate call rendered into a prompt an agent runs carries no `--wait`, so the agent's call takes the capped form and keeps its cache warm, while a call Go runs keeps `--wait`.

`Config.AgentGateCall(key, cards...)` renders `lym gate test --gate <key>` followed by one `--card <id>` per card, refusing as `GateCall` does a key the config cannot run; its doc says it is the form for a call an agent runs, and `GateCall`'s doc says its `--wait` form is for a call Go runs, at a verify site or through `internal/cardverify` from a Go verb.
The agent renders switch to it: `cardGateCommand` (the implementer's card gate), `renderBatchGate` (the batch gate), and the `webster-fix` call that `FixVerifySpawnOrAttach` (the cold fixer) and `Run` (the fixer fork's prompt) render into the `fix_gate` marker.
The Go sites keep `GateCall`: the webster verify gate and webster's verify verb, loom's `gateCallSource` for the landing and fix rows, and a card's `**Verify:**` line, which keeps its required `--wait` since Go re-runs it outside an agent hook.
The websterengine doc's card gate, batch gate and fixer sentences, `contracts/specs/loom-plan-spec.md`'s card and batch gate sentences and the gate-keys sentence of `pattern/PATTERN-test-speed.md` render the agent calls without `--wait` and say a Go site runs `--wait`; the gateslot doc gains `AgentGateCall` beside `GateCall`.
The spec's gate-keys sentence ("every site runs `lym gate test --wait --gate <key>`") says a Go site runs `lym gate test --wait --gate <key>`, and an agent's card, batch and `webster-fix` calls carry no `--wait`.
The spec's `verify-gate-no-wait` reason and the Detail of the `verifyGateNoWaitCheck` finding in `verifyLineFindings` give the reason that holds once only an agent's capped front answers 75: Go re-runs the line outside an agent hook, and a gate call without `--wait` is a slot-held line, run inside one slot under `cardverify`'s per-line timeout instead of taking its tiers' own slots.

Tests, each existing assertion of a rendered agent call dropping `--wait`:
- `TestConfig_TierUse`'s `GateCall` subtest gains the same rows through `AgentGateCall`, without `--wait`, and its refusal of `nope`;
- `TestCardGateCommand`, `TestRenderCardGates_OneBulletPerCardWithAStepSubBulletEach`, `TestRenderBatchGate` and `TestRenameBoundary_BatchesCheckAndGateTheirOwnTree` (tier0), `TestBeginBatch_PromptFile`, `TestRecoverSpawnOrAttach` and `TestFixVerifySpawnOrAttach` (the external package) and `TestRebaselineCmd_EditedCardOfFailedBatchThenRecover` expect `lym gate test --gate card --card …`, `lym gate test --gate batch --card …` and `lym gate test --gate webster-fix`.

**ImpactSummary:** four webster render sites switch to a new sibling of `GateCall`; the Go-run sites, `GateCall` itself and the card Verify rule stay, and the tests asserting the rendered strings follow.
