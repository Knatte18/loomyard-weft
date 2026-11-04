# Reap dobby and orch watch children; tender ignores zombies

Dobby and `lym orch watch` started child `lym` processes and never waited on them.
Each finished child stayed a zombie until its parent exited, and `lym tender status` listed it as a live owned process.
On a long-running hub this piled up to hundreds of defunct `batten` processes under landed slugs.

## What changes

- New `proc.Reap(cmd)` waits on a started command in a background goroutine and discards its exit.
  Every place in `lym` that starts a long-lived child and does not wait on it now calls it after a successful `Start`: `dobbyengine.SpawnDetached`, the `adoptcli` suite start, `fabricengine`, `reedengine` (session and watchdog starts), `boardengine`, `battencli`, `fronttask`, `reedcli`, `termwindow` and `vscode.Launch` on Linux and Windows.
- New `proc.ReapInheritedChildren` handles a process that re-execs itself onto a new binary.
  The re-exec keeps the children but loses the goroutines that waited on them.
  `lym beam dobby` and `lym orch watch` call it once at startup, before they start a child of their own, so every child it finds belongs to the replaced image.
  On a platform that cannot list processes it finds none.
- `runproc.List` and `runproc.Owners` leave out a record whose process is a zombie, so `lym tender status` and `lym tender reap` stop listing landed runs as live.
- `PATTERN.md` gains `PATTERN-reap-children`, and `PATTERN-spawn-observability` and the package docs say the child is reaped.

## Design decisions

- A zombie leader's record stays on disk and is removed once the zombie is reaped.
  Members of the leader's process group may still run, and an end (`End`, `EndSparing`, `EndRole`) must still signal the group.
  The end sequence therefore reads records through `ender.recordLister`, whose running check is the alive check, so a zombie leader stays in its set.
  Only the read paths (`List`, `Owners`) hide it.
- Reaping is a background wait, not a detach, so the starter keeps ownership while it lives; when the starter exits first, init reaps the child.
- `ReapInheritedChildren` must run before the process starts a child itself, otherwise it races that child's own wait.
  In `beamcli` it runs after the worktree is placed and before the first start, so a refused start waits on no child.

## To check by hand

- Every place that starts a child process and does not wait on it calls `proc.Reap` or waits through `cmd.Wait`; a start outside the files listed above would still leave a zombie.
- The startup reap is wired through the `reap` seam of `runDobbyWith` and the `reapInherited` field of `orchCLI`.
  Their tests count the calls, but they do not pin the call's order against `orchengine.LoadState` in `watch`.
- On a hub running the old binary, restart dobby and `lym orch watch`, then confirm `ps` shows no `[lym] <defunct>` children and `lym tender status` lists no landed slug as owned.
  No end-to-end run against a real long-lived hub was done.
- No regression check ran for this task.
