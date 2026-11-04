```yaml
lens: code-correctness
outcome: reported
```

Task: correctness lens, running each changed function or branch on a concrete input from the inherited context.

**Case list**

| Function or branch | Case run | Verdict |
|---|---|---|
| `proc.Reap` (`reap.go:13`) | A started `true` child. The goroutine's `Wait` reaps it. Callers read only `cmd.Process.Pid` and `cmd.Args` afterwards, and `Wait` writes neither. | Correct. |
| `Reap` call sites (boardengine, fabricengine, battencli, adoptcli, reedcli, reedengine, fronttask, termwindow, dobbyengine) | `Start` fails. Each site returns before `Reap`, so no goroutine waits on a nil or unstarted `Process`. | Correct. |
| `Reap` call sites, other waiters | A second `Wait` on the same `cmd`. `spawnPass` (`dobbydeps.go:380`) keeps its own `Wait` and does not call `Reap`. No wrapped site waits twice. | Correct. |
| `Reap` in `fabricengine/spawn.go:100` and `boardengine/spawn.go:57` | Child exits and is reaped before `runproc.ChildRecord` and `Register` read its start time. The record then carries an empty start time. `live()` returns false because `isRunning` is false, so the record is pruned. | Correct. A zombie used to keep the record readable. Now a short-lived child may leave a record that is pruned on its first read. |
| `reedengine.SpawnWatchdog` | `Start` fails. The new `return` skips `Reap`. | Correct. |
| `fronttask.startDetached` | `Release` used to return its error. It now returns `nil`. | Correct. |
| `ReapInheritedChildren` (`reap.go:22`), normal case | Entries with `PPID == self`, one exited and one running. `FindProcess` plus a goroutine `Wait` reaps the zombie at once and the running child when it exits. | Correct. |
| `ReapInheritedChildren`, platform and errors | Windows `ListStats` returns `ErrUnsupported`, so it returns `nil, nil`. Any other `ListStats` error is returned and both callers log it. Pid reuse is not a risk because the pids come from the snapshot of `/proc`. | Correct. |
| `ReapInheritedChildren`, double wait | A child the process started itself before the call. The second `Wait` gets `ECHILD`, which the helper's own goroutine ignores. An owner's `cmd.Wait` could instead get `ECHILD` and return a wrong exit status. The doc states this contract. | See F2. |
| `runDobbyWith` | `placeDobby` refuses. It returns before `reap()`, so nothing is reaped. `placeDobby` itself does git in-process and starts no child. | Correct. |
| `runDobbyWith` | A second dobby that loses the lock. The reap runs first and finds no children. | Correct. |
| `orchcli` watch `RunE` | The reap runs before the lock and before `LoadState`. Only `clihelp.ShouldAbort` runs ahead of it, and that starts no child. | Correct. |
| `lister.live` | A zombie whose stat start time matches the record. `isRunning` is false, so it returns false and the record is pruned. | Correct. |
| `lister.live`, Windows | Start time unreadable and the record's start time empty. It returns true while the pid runs. This is the same as the old `empty && isAlive`. | Correct. |
| `ender.end` (`end.go:141`) | A zombie leader is listed because `isRunning` is `e.isAlive`. `planRecords` makes a group unit and `groupHolds` ignores zombies, so the group signal reaches live members. The existing `end_test.go:190` row covers this. | Correct inside `End`. See F1 for readers outside it. |

**Findings**

- **F1, LOW, design** — `internal/runproc/record.go:187-191` with `internal/pairteardown/teardown.go:193,675`.
  - `runproc.List` now deletes the record of a zombie leader. `End` deliberately keeps such a record (`end.go:140`) so it can signal group members that still run.
  - Any earlier `List` or `Owners` call prunes it first. `lym tender status`, the fabriccli listings and the pairteardown "live records" checks all do. A later `End` or teardown then finds no record, and the still-running group members of that leader are never signalled.
  - Before the diff the zombie record stayed listed, so `End` reached them.
  - The case needs a leader that exited unreaped while members of its group still run. It is rare, but the result is a silent orphan.
  - Evidence: `record.go:185-192`, `end.go:140-141`, `end.go:319-331`.

- **F2, LOW, design** — `internal/beamcli/cli.go:416` and `internal/orchcli/watch.go:99`.
  - The "before any child is started" ordering is a caller convention. `ReapInheritedChildren` snapshots `PPID == self` and does not check that a child is inherited.
  - I could not confirm that nothing earlier in the process starts a child. That covers root and persistent pre-run hooks, and anything in the process image that spawns before `RunE`.
  - If such a child exists, a goroutine `Wait` races its starter's `cmd.Wait`, and the starter gets `ECHILD` with no `ProcessState`.
  - Evidence: `reap.go:19-21` and the unread cobra pre-run chain.

- **F3, LOW, design** — `internal/reedengine/lifecycle.go:467`, outside the diff.
  - `spawnSession` still calls `Start` on the tmux server with no `Wait` and no `Reap`. A long-lived reed process that respawns it, such as the reed watchdog daemon, would pile up zombies. `vscode.Launch` has the same pattern but runs in a short-lived CLI.
  - I did not check whether the watchdog calls `spawnSession`.

- **F4, NIT, consistency** — `internal/runproc/end.go:141`.
  - `lister{isRunning: e.isAlive}` binds an alive check to a field named `isRunning`. The comment explains it, but the name misleads.

**Tool calls:** none. The inherited context held everything the lens needed.
