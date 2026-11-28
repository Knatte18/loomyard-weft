```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I tried to break the record's bounds, guarantees and claims about existing code. Two claims broke, one could not be confirmed, and the rest held.

**Claims attacked**
- D3: the bound "the edge lets an empty `on_done` end the run only inside a block that is the parent's last row; no skip can reach the empty exit". Held. The `always` gate never skips (`internal/shedrecipe/entries_block.go:44-46`), and inlined top-level rows keep the `on_done` they set. Friction-Reflect is the only empty edge (`contracts/recipes/tapestry-recipe.yaml:545-549`).
- D3: the claim that the loom contract's terminal check still binds. Held. `onDonePath` stops at Friction-Reflect's empty `on_done` (`internal/shedbuild/loomcontract.go:157-177`).
- D2: "a run parked inside the Motif block goes on to Preflight and Record-Seed, which writes nothing". Held, because Discussion leaves both files.
- D2: "a run parked at Preflight that never entered Motif goes on to Record-Seed". Held. The undiscussed case is Stuck, and its way forward `goto Discussion-Write` routes back through Discussion-Bouncer to the Jacquard block's exit, then Preflight and Record-Seed again.
- D4: a contract with both a record row and a seed row, where the seed row must follow on the `on_done` path. Held for tapestry: Motif, Discussion-Write, Discussion-Bouncer, Jacquard, Preflight, Record-Seed.
- D4: "the decision verbs treat RecordSeed as they treat RecordWrite". Held. A jacquard run has no approval row and a stored board record, so `boardCopyDue` refreshes the board copy as it does for a raddle-filed darn part (`internal/loomcli/decision.go:697-705`, `:1351-1358`).
- D5: "judged in two places". Broken (finding 2).
- D5, `lym loom start`: "a refused start writes no status file". Held, though the start leaves a seed. `WriteSeed` and the origin record are written before `loomshed.Seed` (`internal/loomcli/sharedbootstrap.go:88-121`), so a refused start leaves an uncommitted seed. The record's way forward removes that seed for a pair batten did not make.
- D5: the batten-site way forward for a slug holding a done motif run: `lym batten cleanup --apply`, the upsert to tapestry, then the run. Broken (finding 1).
- D5: "a jacquard start meets that branch as a raddle start does". Not confirmed. My two reads of `internal/pairteardown` were inconclusive, and the record's own Note says the claim is unverified (finding 3).
- D5, the dobby: its view of a done run reads the same `startClearsMotifRun` (`internal/battencli/dobbyreads.go:491-498`). Held: widening the predicate also covers the dobby.
- D6: "nothing is filed under tapestry". Held for new filings. Parts filed under tapestry before the change keep tapestry and pass over Motif as discussed entries.
- D7: the reinterpretation repairs a gained `defaults.jacquard` before the file must load. Held. The reinterpretations run after the template follow in the same walk (`internal/hubreconcile/walk.go:250-266`).
- D7: "an existing hub's profiles file gains the blocks at its next reconcile". Held as stated, but the result is wrong for a tuned shipped profile (finding 4).
- D9's bound and D8's "a default that falls back to tapestry stays tapestry". Held (`internal/loomcli/sharedbootstrap.go:275-279`).

**Findings**
1. MEDIUM, design, `_lym/discussion/decision-record.md:107-109` (D5, batten-site way forward).
   - Summary: for a slug holding a done motif run, the way forward (`lym batten cleanup --apply`, upsert tapestry, run) does not account for the motif pair's leftover task branch. The new tapestry start would then halt at Worktree-Create.
   - Counterexample: `Cleanup` "never touches a pair, a branch" (`internal/battencli/cleanup.go:113`). `doneSlugRefusal` states "a torn-down pair keeps its task branch locally and on the remote" (`internal/battencli/arm.go:942-947`). Worktree-Create turns a leftover branch into a halt whose way forward is `lym batten cancel &lt;slug&gt; --remove` (`internal/battencli/wire.go:367-393`).
   - Fix: name `lym batten cancel &lt;slug&gt; --remove` from the prime in place of `cleanup --apply`, or state that the branch is discarded, with the evidence for it.
2. LOW, design, `_lym/discussion/decision-record.md:110-112` (D5, "judged in two places").
   - Summary: `lym loom step` also seeds a first status file through `seedAndCommitBootstrap`, so a jacquard seed of an entry with no record passes over the start refusal. Only Record-Seed's Stuck stops it.
   - Evidence: `internal/loomcli/sharedbootstrap.go:65`, "Both `start` and `step` call it".
   - Fix: put the check in `seedAndCommitBootstrap` so both verbs reach it, or name `step` as left to Record-Seed's Stuck.
3. MEDIUM, design, `_lym/discussion/decision-record.md:123` and `:259-261` (D5 and the Notes).
   - Summary: whether a jacquard start after `ClearMotifRun` meets the motif run's task branch is a fact the design rests on. The record leaves it to the plan writer with a conditional fallback to `ClearDoneRun`.
   - Evidence: `ClearMotifRun` passes no branch discard (`internal/battencli/cleanup.go:137-143`), while `ClearDoneRun` passes `discardTaskBranchIfPresent` (`:157-168`).
   - Fix: do the research and record the answer with its evidence. Pick one clear path.
4. MEDIUM, design, `_lym/discussion/decision-record.md:143-151` (D7).
   - Summary: on a hub that tuned a shipped profile's tapestry block, or `defaults.tapestry`, the follow adds the shipped template's jacquard block and `defaults.jacquard: solid`. Raddle parts that used to run with the operator's tuned tapestry settings would silently run with shipped jacquard values.
   - Evidence: `followUnit` adds a template value for a key that both base and file lack (`internal/yamlengine/follow.go:167-181`). The reinterpretation covers only operator-added profiles.
   - Fix: build the jacquard block of every profile holding a tapestry block from that block minus `loom.discussion`, and derive `defaults.jacquard` from `defaults.tapestry`. Or state the reset as accepted.

This round admits a new finding only at BLOCKING and only on a changed hunk. Findings 1, 3 and 4 sit on hunks this round changed (D5 and D7); leave their severity to the handler.

**Tool calls**
- `internal/battencli/dobbyreads.go` start-refusal view: to check whether the dobby reads the done-motif clear, which is one hop past batten.
- grep of `boardRecordStored`, `pastApproval` and `storeRecord` in `internal/loomcli/decision.go`: their uses were outside the read spans.
- `internal/loomcli/decision.go:688-730` and `:1340-1375`: `boardCopyDue` and the store verb, to test "treat RecordSeed as RecordWrite".
- grep of the teardown packages for task-branch handling: to test the D5 branch claim (inconclusive).
- grep of "branch" in `internal/pairteardown/doc.go`: the same claim (inconclusive).
- grep of "branch" in `internal/pairteardown/teardown.go`: the same claim (no match).
