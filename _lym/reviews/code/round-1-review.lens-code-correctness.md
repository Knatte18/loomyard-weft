```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens, covering each function or branch the diff changes, the case I ran, and whether it is correct.

**Per function or branch**

- `taskWorktreePresent`, missing path: stat gives not-exist, so it returns `false, nil`. Correct.
- `taskWorktreePresent`, non-not-exist stat error: it returns the error. Correct.
- `taskWorktreePresent`, stray directory or plain file: stat succeeds, `ResolveWorktree` fails, and it returns `false, nil`. Correct for the goal.
- `taskWorktreePresent`, real worktree: it returns `true, nil`. Correct.
- `taskWorktreePresent`, `ResolveWorktree` failing for any other cause: the cause is discarded, so a transient failure reads as absent (finding F1).
- `hubReadoptSeams.Gone`: it returns `!present` together with the error, and the caller in `ownerPreflight` checks the error before the bool. Correct.
- `ownerPreflight` at Worktree-Teardown with a stray directory: `present` is false, so it judges the owner on the local board and returns nil. Correct.
- `ownerPreflight` at Worktree-Create with a stray directory: `present` is false, so it takes the start-owner path instead of failing. Correct.
- `Promoted`, slug absent from the board: it returns false. This is the beam's own landing after Land pruned the entry. Correct.
- `Promoted`, slug a project or note: it returns false through `!entry.IsTask`. Correct, same as the old project case.
- `Promoted`, slug a task whose pair is marked: it returns true. Correct.
- `refuse`, promoted with `ReachedParent` returning true: it passes. Correct.
- `refuse`, promoted with `ReachedParent` returning false: it refuses and names `cancel --remove`.
  - `cancelRun` calls the same `hubChildReachedParent` probe with the same status.
  - The cancel therefore also reads "not reached" and goes ahead, so the loop is gone. Correct.
- `refuse`, promoted with a `ReachedParent` error: it returns `probeRefusal` naming the probe and the rerun. Correct.
- `refuse`, promoted with `found=false`: `childReachedParent` reads only the child, so a zero status is safe. Correct.
- `refuse`, batten status at Worktree-Teardown with a promoted task: `childReachedParent` reads true, so it passes. Correct.
- `refuse`, absent board entry plus a marked pair and a fresh start (no status):
  - `Promoted` now returns false.
  - The `PairParts` left-part check still refuses naming `cancel --remove`.
  - Correct, no hole.
- `hubChildReachedParent`: the `battenCLI` it builds uses only `location` and `slug`, which `readChildStatus` needs. A stray directory makes `readChildStatus` return not-found, so `FinalizePastMerge` is skipped and no bare "not a git repository" error escapes. Correct.
- `hubCancelDeps`: the `ChildReachedParent` refactor is behavior-identical. Correct.
- Cancel at Worktree-Teardown over a stray directory: it refuses with "reached its parent, run `lym batten run`". The run then passes the guard and goes on through `ownerPreflight`, so there is no loop. Correct.
- `taskWorktreeComplete` with a stray directory: `present` is false, so it reaches `Topology.Add`, which refuses with `ErrTaskPathTaken` and names the path. Correct.
- `ReleaseRunParentLock`, `beamfork`, `readNoticeChild` and the DriverStrand/ArbiterInFlight rows: they all gate on `taskWorktreePresent`, so a stray directory reads as absent and none of them reaches `taskWorktreeLocation`. Correct.

**Findings**

- F1, LOW, design, `internal/battencli/wire.go:265-267`.
  - `taskWorktreePresent` discards every `ResolveWorktree` error.
  - A real worktree whose anchor file is unreadable (permissions, transient I/O) reads as absent. Resume then runs `readoptGonePair` (prune, then `Add`) instead of surfacing the cause.
  - `Add` should refuse it by name, so the damage is bounded.
  - The summary records this as a deliberate decision, so it is a note rather than a defect.
- F2, LOW, consistency, `internal/battencli/cli.go:69`, `internal/battencli/wire.go:208-229`.
  - The doc says a stray task worktree path is gone and "every batten verb reads it so".
  - `taskWorktreeLocation` and `presentTaskWorktreePath` still test only that the path exists. For a stray directory they return the path and then fail with the resolver's generic "not a git repository" error.
  - Callers: `goto.go:19` (error read as "child not done") and `readChildStatus` (error swallowed), both tolerant.
  - `taskWorktreeLocator` (`wire.go:234`) is reached after `ownerPreflight` has already routed the stray case.
  - No reachable path fails today, but the "one answer" and "every verb" claim is stronger than the code.

Tool calls: none were needed, since the inherited context held every definition and caller I used.
