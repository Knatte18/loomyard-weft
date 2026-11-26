```yaml
lens: interactions
outcome: reported
```

Interactions lens: I checked the pairs of the record's own decisions and the criteria and constraints that touch each pair.

**Findings**

1. **MEDIUM — D12 against D5, D6 and D9 (sequencing).**
   - **Summary:** D12 makes the foundation part do every cross-module move, but its "goes when the core, action or seam tests of D6 hold its assertion" route and its "rewritten over fabric's own … snapshot" route need tests and snapshots that only the later concern parts create.
   - **Why it conflicts:** D5 lets a test be deleted only in the change that adds the test holding its assertion. D9 part 1 has no concern moved, so no test can go there, and none can be rewritten over a snapshot.
   - **Evidence:** many of the cross-module files are flow tests of concerns that later parts own. They import shedengine, shedrun, boardengine, heddleengine, stencilstore, hubreconcile or fabriccli. Part 2 owns `originreconcile_integration_test.go`, `originreconcilemoves_integration_test.go` and `originreads_integration_test.go`. Part 5 owns `replay_integration_test.go` and `boltreplayers_integration_test.go`, and the `livestate_verbs_test.go` matrix spans several concerns. In the foundation part these can only move whole to another module or be rewritten over today's code. D6 then converts the same flows again later.
   - **Fix:** say that each concern part makes the cross-module moves of its own concern, with the guard landing in part 7. Or say that the foundation part only moves or rewrites, without dropping, and later parts drop.
2. **MEDIUM — D12 against the `PATTERN-fabric-boundary` vocabulary rule (Constraints).**
   - **Summary:** tests D12 moves into shedengine, boardengine, heddleengine and the other non-owner packages carry bare weft and warp tokens. `TestEnforcement_FabricVocabulary` refuses those tokens in any `_test.go` file outside the owner set.
   - **Evidence:** side-token counts in those files include `livestate_verbs_test.go` (88), `replay_integration_test.go` (67), `originreconcilemoves_integration_test.go` (35) and `rejectedtakeover_integration_test.go` (21). The record says nothing about this.
   - **Fix:** D12 states that a moved test drops the side spelling, or that the test stays a fabric test rewritten per D6.
3. **MEDIUM — D4 against `PATTERN-fabric-boundary` (destruction chokepoint).**
   - **Summary:** D4 has the generic executor record every executed action in `Mutations`. The entry says "The `rec *Mutations` recorder is threaded into `destroy.go` only". `## PATTERN changes` does not change that clause, so the departure is silent.
   - **Evidence:** the clause is already stale in the code. `*Mutations` appears in about 50 production files of fabricengine, among them `anchorchange.go`, `junction.go`, `scratchlink.go`, `reconcile.go` and `remove.go`. The record should still name the clause and change it, not inherit the drift.
4. **MEDIUM — D2 against D3 (lock scope of the re-read).**
   - **Summary:** D2 binds each snapshot read to the verb's lock "at the point it takes it". D3's loop re-reads the stale fields after an outcome without saying under which lock, or whether the lock is held across rounds.
   - **Why it matters:** D4 says locks are held "exactly where they are today", but today's push retry re-reads origin mid-flow under the remote-refs lock. If D3's re-read is a fresh snapshot read taken outside that hold, it weakens D2's read-then-act guarantee.
   - **Fix:** D3 states that the loop's re-reads run under the lock the verb holds at that step today.
5. **MEDIUM — the D2 criterion against D10 and the Goal.**
   - **Summary:** "for each moved verb … including the fallback to git for a store whose ref storage is not `files`" asks for a non-`files` ref-storage repository for every verb's snapshot test.
   - **Why it conflicts:** that multiplies the real-repository fixtures that D10's target and the Goal aim to cut. The fallback belongs to the shared reader path, so one test is enough.
   - **Fix:** one tier1 test of the fallback, with the other snapshot tests on the `files` store.
6. **LOW — D1's rationale against D8 and D12.**
   - **Summary:** "only `internal/fabricengine` imports `internal/fabriccore` (new)" is false once D8's `internal/testkit/fabrickit` (new) and D12's allowed test imports exist.
   - **Why it is low:** the gate-scope conclusion still holds, because only fabric's own test packages import the kit.
7. **LOW — D1 ("every rule" in the core) against D4.**
   - **Summary:** the execution-time refusals in `destroy.go` (containment, ownership, dirtiness) stay in the engine, as do its git-reading ownership probes.
   - **Fix:** D1's "every rule" should except D4's execution-time guards so a plan writer does not try to move them.
8. **LOW — the D12 import set.**
   - **Summary:** "the tmux kit's `Main`" names no package, and the tier0 guard needs an exact closed set. The set also omits any `lymbin` or `envkit` use that a kept tier2 test, such as the killed-push test, may need.

**Pairs checked**

| Pair | Result |
|---|---|
| D1 and D7 (layout holds names and types; retarget, no aliases) | Held. The option and result types reference only `time`, so they move cleanly. |
| D1 (layout does no I/O) and D7 (name-only callers) | Held. Callers of I/O helpers stay on the engine. |
| D1 and D8 (only the engine imports the core; the kit imports the core) | Broken, but only in wording (finding 6). |
| D1 and D4 (every rule in the core; guards in the engine) | Imprecise (finding 7). |
| D1 and the `PATTERN-leaf-packages` change | Held. |
| D2 and D3 (lock on the snapshot read; loop re-read) | Gap (finding 4). |
| D2 and D4 (stale snapshot; execution-time checks) | Held. |
| D2 and D10 (no more git processes; target) | Held. The criterion conflicts (finding 5). |
| D3 and D4 (loop rounds; rollback over executed actions) | Held. |
| D3 and the D3 criterion (retry rounds equal today's) | Held. |
| D4 and the `PATTERN-fabric-boundary` destruction chokepoint | Silent departure (finding 3). |
| D5 and D7 (no behaviour change; mass retarget in the foundation part) | Held. Only imports move. |
| D5 and D12 (delete only with a replacement; foundation-time drops) | Broken (finding 1). |
| D6 and D12 (flow tests converted per concern; flow tests moved out in the foundation part) | Broken (finding 1). |
| D7 and D9 (retarget lands in part 1) | Held. |
| D8 and D12 (kit in the allowed set) | Held. |
| D8 and `PATTERN-test-isolation` (kit import limits) | Held. |
| D9 (serial chain) and raddle's `packages-shared` check | Held. |
| D9 part 2 (push actions) and part 3 (archive-tag pushes in teardown) | Held. |
| D9 and D10 (close-out in part 7) | Held. |
| D11 and the Goal, the D6 wording and the `PATTERN-fabric-core` text | Held. |
| D12 and the vocabulary owner set | Broken (finding 2). |
| D12 and `internal/hubforge` reaching `internal/fabriccli` | Held. The record states it. |
| The `PATTERN-fabric-boundary` change and D1 (fabric's membership) | Held. |
| `PATTERN-cwd-resolution` and D2/D3 (anchors read and written by the engine only) | Held. |

**Tool calls**
1. A Bash grep of the fields of `AddOptions`, `SyncOptions`, `ReconcileScope`, `OriginReport`, `RemoveResult` and `MergeResult`, plus a count of `*Mutations` uses per production file. My context did not show whether the types D1 moves carry non-leaf types, or how far the recorder reaches.
2. A Bash count of side tokens in the fabricengine test files that import another module's package. My context did not show whether D12's moves would trip the vocabulary scan.
