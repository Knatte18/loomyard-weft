```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens report: each decision run on one concrete case, with the end state checked against the decision's purpose.

**Findings**

1. **MEDIUM, D2 and D4: `LivePair` moves from move time to snapshot time, and no execution-time recheck replaces it.**
   - Case: `ReconcileOrigin` moving an ended entry's branch.
   - Today `originreconcilemoves.go` calls `m.seams.LivePair(slug)` inside the moves, just before acting on that slug.
   - D2 folds "the facts a verb's seams supply … the live-pair probe" into a snapshot read once under the verb's lock.
   - The remote-refs lock does not stop a reed session or run lock going live. A pair that goes live between the snapshot and its move is then moved, which today's code refuses.
   - D4 keeps only `destroy.go`'s checks and the locks at execution, so nothing rechecks liveness.
   - Fix: list live-state probes (`LivePair`, `PrimeParentLockFree`) as execution-time checks the engine runs before the action, or require the core to mark them stale before each move action.

2. **MEDIUM, D3: the `Teardown` seam is an effect, and the action kinds have nothing for it.**
   - Case: `originreconcileended.go` hands an ended entry to `seams.Teardown` mid-reconcile.
   - The `TeardownReport` it returns then feeds later verdicts.
   - D3's action list holds only git and filesystem effects (refs, tags, pushes, worktrees, anchors, excludes, commits, pending records). No kind covers "run a caller-supplied seam and feed back its report".
   - The walk stalls at that step. Either the core cannot express the teardown, or the engine keeps a rule about when to call it, which goes against D3's "engine stays a loop with no rule of its own".
   - Fix: add a seam-call action kind whose outcome feeds the loop like a push answer does.

3. **MEDIUM, D12: going through `internal/hubforge` brings in far more than `internal/fabriccli`.**
   - Case: a change to `internal/hubreconcile`, `internal/loomengine` or `internal/configengine`.
   - `internal/hubforge` imports `fabriccli`, `configengine`, `configreg`, `hubreconcile`, `loomengine`, `clihelp`, `fslink` and `testkit/envelope`.
   - `impactset` counts a test import's non-test dependencies, so each of those changes still pulls fabricengine's tests into scope.
   - D12's rationale names exactly this harm ("puts fabric in that module's gate scope"), and only the `fabriccli` path is acknowledged as allowed.
   - Its guard counts direct imports only, so the end state misses the stated purpose. This also bears on the operator's "KUN fabricengine".
   - Fix: name every package `hubforge` brings in and accept them as the mandated fixture's cost, or decide to slim `hubforge`'s import set. Since this touches the operator's intent, raise it to the parent.

4. **LOW, D10 and D12: the remeasure can count moved time as saved.**
   - Case: the foundation moves about 25 cross-module tests, with their hubs, into `shedrun`, `shedengine` and similar packages.
   - D10 measures only `fabricengine` plus `fabriccore` tier1 time, so the 60 s target can be met partly by moving time elsewhere.
   - Fix: also record the whole tier1 run time, or the receiving packages' added time, against the 250 s baseline.

5. **LOW, D1: `internal/fabriclayout`'s leaf rule may not hold for every option type it is given.**
   - Case: `SyncOptions` moved to the layout.
   - It carries a `Notifier PushNotifier` field, an engine-declared interface, and `AddOptions` carries function fields.
   - If those field types refer to engine or non-leaf types, moving them into a leaf package that "does no I/O" and imports only leaves either fails or drags those types along.
   - The record should state whether an option type whose fields name engine seams stays in the engine.

**Decisions walked**

| Decision | Case | Purpose met? |
|---|---|---|
| D1 | A caller using `RecordsBranchName` only, which imports the layout | Met. Option and seam field types are open (finding 5). |
| D2 | `ReconcileOrigin`'s snapshot | Partly. Live-state probe timing is lost (finding 1). |
| D3 | Push rejection, then re-read origin, then the core retries within today's classes | Met. Seam-effect step has no action kind (finding 2). |
| D4 | Destroy of a worktree that turned dirty after the snapshot; `destroy.go` refuses | Met. `Mutations` is already threaded through many files, so the record's "executor records" claim is consistent with today. |
| D5 | Replacement and deletion in one change | Met. |
| D6 | `TestCrossProduct` split into core rows, action tests and a seam test | Met. |
| D7 | A board package using `BoardDir` only now skips engine-only changes | Met. |
| D8 | Kit builds snapshots; imports stay clean | Met. |
| D9 | Foundation, then push and origin; serial `depends_on` satisfies `packages-shared` | Met. |
| D10 | Remeasure | Partly (finding 4). |
| D11 | Naming | Met. |
| D12 | A `shedrun`-driving test moves to `shedrun` | Direct imports met; gate-scope purpose missed (finding 3). |

**Tool calls**

1. Read `hubforge`'s imports, `AddOptions`'s fields and which production files take `*Mutations`. My context lacked `hubforge`'s import set (for D12) and the option field types (for D1).
2. Read the `OriginSeams` declaration, the call sites of `LivePair` and `Teardown`, and `SyncOptions`'s fields. My context had only the subagent's mention of the seams, not when they are called (for D2, D3 and D4).
