# Card 9 — `lyx orch start`, `lyx loom start` and `lyx loom resume` reconcile first

**Edit:**
- `internal/orchcli#Command`
- `internal/loomcli#loomCLI.arm`
- `internal/orchcli/doc.go`
- `internal/orchcli/refusal_integration_test.go`
- `internal/loomcli/smoke_prebootstrap_integration_test.go`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `internal/hubgeom#ReconcileGeometry`
- `internal/hubreconcile#Ensure`
- `internal/hubreconcile#Options`
- `internal/loomcli#StartAliasCommand`

**Intent:**
Decision 3: each start verb calls `hubreconcile.Ensure` before any of its own work, and only a start verb does.
"Before any of its own work" means before its pre-run loads a module config, since a strict config load of a file carrying a retired key would otherwise refuse the verb before the reconcile that removes the key could run.

`orchcli`'s `Command` pre-run, for `cmd.Name() == "start"` only, right after the prime-only refusal (`refuseNonPrime`) and before `orchengine.LoadConfig`:
- When `hubgeom.ReconcileGeometry(location)` reports a hub, it calls `hubreconcile.Ensure(geom, hubreconcile.Options{})`.
- A returned error goes through the pre-run's existing `fail`, so `lyx orch start` stops with a JSON error envelope carrying the error's own message and way forward (`PATTERN-cli-cobra`).
- `status`, `watch`, `stop`, `refresh` and `distill` never call it.

`loomCLI.arm`, for `verb` `start` or `resume` only, calls it the same way right after `lyxcwd.Resolve` and before `armAt`, returning the error.
That covers `lyx loom start`, `lyx loom resume` and the root `lyx start` alias, whose `StartAliasCommand` attaches the same pre-run.
`ArmAt` (the `lyx shed` path) and every other loom verb never call it.

On a fresh stamp the call costs one file read, so existing integration tests driving these verbs on hubforge hubs see one walk at their first start, which writes no config, lands no commit and only writes the stamp under `_board/.lyx/config/`.

`contracts/specs/refusal-spec.md`'s loom section gains two rows, since both refusals are reachable through loom verbs (`PATTERN-refusal-way-forward`):
- "start/resume: hub config reconcile failed": a start verb's walk fails on a config file or a commit. Class: correctness halt. Way forward quotes the message: fix the named file (or run `lyx config <module>` on it), then re-run this command.
- "start/resume: hub reconcile lock busy": another command holds the hub's reconcile lock past the wait. Class: transient. Way forward: re-run this command once it finishes.

`orchcli/doc.go` gains one paragraph saying `start` reconciles the hub's config after a binary change before launching anything, and that no other orch verb does.

Tests:
- `refusal_integration_test.go` gains a step on its hub that writes an unparseable `loom.yaml` into the pair's anchor and removes the stamp.
  `start` from the prime then exits 1 with an envelope naming that file and `lyx config loom`, launches nothing (no `.lyx/orch` under the prime), and leaves no stamp.
  `status` on the same hub writes no stamp.
  The step runs last and restores the file.
- `smoke_prebootstrap_integration_test.go`, which already drives the built binary's loom verbs on a hubforge pair before bootstrap, gains one test on a `newWiredPairFixture` pair whose committed records-side `batcher.yaml` carries the retired `master_base` key and whose hub has no stamp.
  `loom status` leaves the key and writes no stamp; `loom resume` then removes the key, leaves the pair's records worktree clean and writes the stamp, whatever its own exit, since the gate runs before `armAt`.
- `TestLoomFailedReedUpRefuses`'s `requireStartRefusal` also asserts the hub's stamp exists after `loom start` refused, which pins the `start` gate on the same fixtures: the reconcile runs before reed comes up.
- The failing-walk envelope is pinned by the `refusal_integration_test.go` step above and the error's text by card 7's `ensure_integration_test.go`, so no loom test repeats it.

**ImpactSummary:** Two pre-runs gain one gated call each: `lyx orch start`, `lyx loom start`, `lyx loom resume` and `lyx start` may now walk the hub's config once per binary change; every other verb is unchanged.

**Commit:** 9: orch, loom: the start verbs reconcile the hub's config after a binary change
