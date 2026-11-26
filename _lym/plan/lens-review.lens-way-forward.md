```yaml
lens: way-forward
outcome: reported
```

Way-forward review: of the refusals, stops and recovery steps the cards add, two next steps fail from the state they leave and two need a missing clause to be fixed.

1. **MEDIUM** (Card 2, `load` default-layer refusal; also Card 9's fallback): the next step is refused when an agent session receives it.
   - The refusal names `lym config reconcile --layer default --apply` from the prime.
   - After Card 9, an agent's own loom verb in a task worktree is the usual receiver.
   - From a session, `runReconcile` → `reconcileDefaultLayer(l, out, apply, realHubWriteGuard(l))` refuses that command (`internal/configcli/configcli.go` runReconcile; `realHubWriteGuard` reads the strand env).
   - The existing board-dir refusals end with `boardDirWayForwardClause` ("a session lym refuses the verb from reports status: FAILED and the orch runs it"): `internal/configengine#missingBaseError`, `internal/configengine#boardDirWayForwardClause`.
   - Card 2 prescribes the bare command with no such clause, so the agent is stranded (PATTERN-refusal-way-forward, "No denied recovery").
   - Fix: append `boardDirWayForwardClause` to the default-layer moved-key refusal, and assert it in the `TestLoad_Refusals` row.

2. **LOW** (Card 2, `Set` refusal): the way forward can print a command that writes the wrong layer.
   - The text "with the layer flag the operator gave kept by the configcli caller's own text" is carried by no card: Card 2 lists no configcli member, and Card 5 lists only `TestDispatchSet`.
   - `configengine.Set` does receive `layer`, so it can render `--layer local` / `--layer default` itself.
   - As planned, `lym config loom --layer local --set code_max_bounces=2` would answer `lym config loom --set code.max_bounces=2`. Repeating that writes the tracked worktree file, a different layer, instead of the local override.
   - Fix: have `Set` render the layer flag from its `layer` argument for a non-base layer, and add a `TestSet` row for a local-layer old key.

3. **LOW** (Card 2 local and worktree refusals → `lym config reconcile --apply`): the recovery leaves a dirty tracked file.
   - The command moves the key; verified in `runReconcile`, which reaches `configsync.ReconcileAll(..., PruneUnknown: true)`, so the move runs before the prune.
   - But it never commits, so in a task worktree the moved `_lym/config/loom.yaml` stays modified. Rows that halt on a dirty tree (Parent-Align and Finalize, per `loom-plan-spec.md`'s verify model) would then stop.
   - The walk commits its moves (Card 6); the operator's route does not, and no card says what follows.
   - Fix: name the commit in the worktree-layer way forward, or have reconcile's envelope say the file needs committing.

4. **LOW** (Card 8, carry failure): the failure names no next step and is reported under the wrong cause.
   - `carryMovedRunSettings` logs a Warn and returns skipped. `ensure` then logs only "the hub-wide config was not committed and pushed or a worktree is mid-merge, so the build stamp stays stale and the next start verb retries" (`internal/hubreconcile#ensure`). That cause is wrong for a carry failure and still names start verbs only.
   - With Card 9, a persistent carry failure makes every loom verb rerun the whole walk (hub lock and board pull), while no warning or way forward reaches the orch.
   - Fix: give the carry's Warn a `way_forward` ("fix the cause the error names; the next verb that loads loom or tapestry config retries the carry"), and widen `ensure`'s skipped message to name the carry.

5. **LOW** (Card 5, `PrepareRunSettings` refusal through `lym board upsert` and `lym config profiles`): no way forward clause.
   - The refusal comes back from `boardcli#validateRunSettings` with none. D4 allows this because the text names the new full name and the operator repeats the write.
   - Card 5 asserts "holding no `way forward:`" only for the `PrepareRunSettings` row. No row checks that Seed-Child's wrapper (`battenshed#seedChildProducer.writeRunSettings`) then prints exactly one way forward, which is D4's "Seed-Child's refusal prints one way forward" criterion.
   - That criterion goes untested. Fix: add a Seed-Child row (battenshed/battencli `TestPrepareRunSettings`) asserting a single `way forward:`.

Tool calls:
- `internal/configcli/configcli.go` setModule, setChecked, runReconcile and realHubWriteGuard: to confirm whether each way forward's command is refused from a session and how reconcile runs. My context held only the function list.
- `grep` of run_settings in `internal/boardengine/task.go` and `store.go`: to check how an upsert replaces `run_settings`. Not in context.
