```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set. For each changed way-forward message, I list what reads or asserts it and whether the diff leaves it right.

**Per change**
- **`lymVerbDir` in `pushStep`, `codeDivergedWayForward` and `rewoundWayForward`:** covered. The existing rows at `failedpush_test.go:223-303` have no marker, so they still name the root. The four new anchored rows pin the anchored paths. `bolt.go:428` goes through `pushStep` with the bolt side, which is unaffected.
- **`stopWayForward` `BranchRewritten`:** not covered. The code side now names the anchored directory, but the records side still names `b.Worktree`. `originreconcile_integration_test.go:495` asserts `row.worktree` verbatim for both sides, so it would not catch the records case.
- **`anchorStopWayForward` (`originreconcile.go:427,429`) and `dirtyWayForward` (`:440`):** the code is converted. `originreconcilemoves_integration_test.go:234,257,263,328` assert the unanchored root paths, so they still pass. No test uses an anchored hub for these lines.
- **`pull.go:442` (`recordsConflictWayForward` push step):** the code is converted. `pull_integration_test.go:406` asserts `fx.warpPath` in an unanchored fixture, so the new `lymVerbDir` call has no anchored row.
- **`doc.go` addition:** it says "Every way forward names a `lym` verb in the code worktree's anchored directory." The set below shows that claim does not hold.

**Findings**

1. MEDIUM, scope. `internal/fabricengine/originreconcile.go:386`. The records-side `BranchRewritten` way forward is `lym fabric pull` in `b.Worktree`.
   - On the records side, `b.Worktree` is the records sibling (`originreconcile.go:724`: `worktree: sibling, codeWorktree: worktree`).
   - The comment at `originreconcile.go:434` says the records sibling refuses every fabric verb, so the line is wrong as written in any hub.
   - The step should use `lymVerbDir(b.CodeWorktree)` for the records side.
   - Pin it with a records-side `BranchRewritten` row, ideally in an anchored hub.

2. MEDIUM, scope. `internal/fabricengine/pull.go:461-462`. `weftDirtyStop` names `lym fabric commit` and `lym fabric pull` in `f.warpPath`, which is the unconverted root.
   - It sits in the same function family as the converted `pull.go:442`.
   - It is refused in an anchored hub, the same defect as the issue.
   - Wrap it in `lymVerbDir(f.warpPath)`.

3. MEDIUM, scope. `internal/fabricengine/anchorstop.go:170-171`. `upstreamAnchorStop` names `lym fabric anchor … --set` in the `codeWorktree` argument as given.
   - `originreconcile.go:427` converted the same verb, so the two disagree.
   - Wrap it in `lymVerbDir(codeWorktree)`, or have the callers pass the anchored directory.
   - I did not read the callers, so they may already pass it.

4. LOW, consistency. `internal/fabricengine/failedpush.go:289-292`. The `PushFailureGone` way forward names `lym fabric remove` "run from the prime", with the pair path as an argument. The directory it runs in is not the prime's anchored directory.
   - I did not read the verb's own cwd requirement, so this is unproven.
   - Check it, and convert it or say the prime's anchor applies.

5. LOW, consistency. The doc and commit message claim too much.
   - `doc.go:1462` ("Every way forward names a `lym` verb in the code worktree's anchored directory") and the commit message ("every lym step now names that directory") do not hold.
   - Counter-examples: findings 1 to 3, and the `lym fabric checkout` lines in `addparent.go:66,72` and `addadoptonly.go:70`.
   - Those last lines belong to add-time refusals, outside the failed-push scope. Either convert them, or narrow the doc and message to the failed-push and reconcile lines the diff touches.

Nothing is wrong on the PATTERN side. `lymcwd.NamedWorktree` from `fabricengine` is allowed, `PATTERN-fabric-boundary` is untouched, and the diff has no `Pattern-Departure` trailer and no declared PATTERN changes. The `doc.go` addition sits in `fabricengine/doc.go`, which is the package's own doc.

**Tool calls (1)**
- One grep over tests and docs for the changed way-forward strings, because my context did not show the existing assertions.
