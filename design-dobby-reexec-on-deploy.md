# Dobby dies at a deploy instead of re-executing onto the new binary

`dobby-reexec-on-deploy` · Note · bug, workspaces, batten

Reported as [issue #880](https://github.com/Knatte18/loomyard/issues/880).

## Seen (OSL-1046, 2026-10-26)

The landing of `beam-start-and-ownership` (e6fbc14c) ran the hub's deploy, which replaced the lyx binary at 15:17:37Z.
The prime's dobby log ends at 15:17:25Z, its process (pid 96674) is gone, and `lyx beam status` reports the prime's dobby not running.
Its state record still holds the board-tidy agent it spawned at 15:17:39Z, with a deadline, so the dobby died without ending cleanly, and the tidy agent's reed window stayed open until the orch removed the strand by hand.
The cause is not found: the re-exec onto the new binary failed, or the landing or teardown ended the dobby's process.

## Expected

A dobby re-execs onto a changed lyx binary between ticks, as `internal/dobbyengine` documents, and goes on running through every deploy, so a landing never leaves the hub without its dobby.
A failed re-exec is logged with its error in `dobby.log` and posted to the orch, never a silent exit.
A test replaces the binary under a running dobby and asserts the dobby re-execs and keeps its tidy in flight.

## Cause (investigation, 2026-10-26)

### How the re-exec works

`internal/beamcli/dobbydeps.go:102` wires `CheckBinary: selfexec.New().Check`, and `internal/dobbyengine/loop.go:231` calls it once per loop iteration, after `tick`.
`selfexec.New` records the executable path and its identity (device, inode, size, mtime) at startup, and `Check` (`internal/selfexec/selfexec.go:71`) execs only when the identity differs on two consecutive checks.
The exec is `syscall.Exec(path, os.Args, os.Environ())` (`selfexec.go:77`, `identity_unix.go`), so the argv is the dobby's own `lyx beam dobby <worktree-root>` and the pid is kept.
A failed exec logs at Warn (which reaches `dobby.log` through stderr) and the process keeps running; a failed exec never exits the dobby.
Neither the dobby's command line nor `internal/selfexec` changed between 744e7842 and e6fbc14c: `hidden dobbyCmd`, `SpawnDetached` (`internal/dobbyengine/start.go:274`, argv `beam dobby <root>`) and `runDobby`'s own head are unchanged, and `git diff 744e7842 e6fbc14c -- internal/selfexec` is empty.
The new binary therefore reads the old argv exactly as before.

A re-exec'd process runs `runDobby` from the top (`internal/beamcli/cli.go:373`): `placeDobby`, then a `dobbyengine.Live(geom)` pre-check (`cli.go:378`), then `registerDobby` (`cli.go:381`), then `dobbyengine.Run`.
The pre-check prints `{"already_running":true,"ok":true}` and returns when the dobby lock reads held (`cli.go:379`).

### Evidence on disk

- `.lyx/beam/markers/dobby.log` ends with the line `{"already_running":true,"ok":true}` right after the `2026-10-26T15:17:25Z pass beam-start-and-ownership continue` line; its mtime is 16:17:47.884 local (15:17:47.884Z).
  That line is `runDobby`'s output (`cli.go:379` or `cli.go:390`), written to the log by a `lyx beam dobby` process: no `{"stopped":true}`, no Warn line and no panic text follows it.
- `.lyx/beam/state.json` was rewritten at 16:17:47.871 local, 13 ms before that line, and still holds pid 96674 and `start_time` 08:50:36Z, so the pid never changed and the last act of the old image was a state write.
- The tidy strand's `.lyx/shuttle/09a6ea13e51eed505fb7b77cd4337e21/run.json` has `"pid": 96674` and `"outcome": "running"`, and was last written at 16:17:43; the dobby that started the tidy at 15:17:39Z was alive until at least 15:17:43Z and never settled the run.
- `.lyx/beam/dobby.lock` is a 0-byte file and no process holds it now (every `/proc/*/fd` scanned).
- The dobby's run-process record `_board/.lyx/runproc/_board/96674-872132.json` (role `dobby`, owner `_board`) was last written at 15:07:11 local (`registered_at` 14:07:11Z), which is the previous successful re-exec onto a deployed binary: `registerDobby` rewrites it on every start.
  It was not rewritten at 15:17:47, so the process that printed `already_running` never reached `registerDobby` (`cli.go:381`), and the exit was the `Live` pre-check at `cli.go:378`, not the 250 ms grace in `Run` (`internal/dobbyengine/loop.go:118`).
- The dobby had survived nine earlier deploys today (selfexec log lines for the overview and the orch watcher at 11:25, 11:42, 11:53, 12:01, 13:19, 13:39, 13:54, 14:20 and 15:07 local, and `dobby.log` pass lines continue after each), so re-exec works in general.
- The landing's own trace `.lyx/logs/trace-20261026T151740Z-0e4db64b354f8952-2102222.log` shows the teardown at 16:17:41: `runproc: ended the processes of an owner owner=beam-start-and-ownership ended=0 survivors=0`, and nothing that signals any process afterwards.
- `lyx batten overview` (pid 92111) and `lyx orch watch` (pid 92464) re-exec'd at 16:17:40.253 and 16:17:41.3; both keep running with new images, and both probe the dobby lock (`internal/battencli/dobbyreads.go:327` through `dobbyengine.Live`, `lockHeld` at `dobbyreads.go:197`).
- The dobby has no trace file in `.lyx/logs` at all (no `-96674.log`), so its own `selfexec: the binary changed on disk` Info line, and any Info line it wrote at the end, is not on disk.
- `journalctl` and `dmesg` are closed to this user (not in `adm` or `systemd-journal`), and `~/.local/state/lyx` holds only per-hub hash directories with no entry for the event, so a kill signal or an OOM kill cannot be excluded by the logs.
- Timeline: deploy replaced the binary 15:17:37.9Z; the dobby's tick at about 15:17:39Z spawned the tidy and saw the changed binary once; the tick at 15:17:47.87Z wrote state and saw it a second time, and exec'd; the new image printed `already_running` at 15:17:47.88Z and exited.

### Most likely cause

The re-exec'd dobby exits at the `dobbyengine.Live` pre-check in `runDobby`, `internal/beamcli/cli.go:378-379`, because it reads the dobby lock as held.
That pre-check is a bare `TryAcquireWriteLock` (`internal/dobbyengine/start.go:243-255`) with no `lock.ProbeGrace` wait, whereas `Run` waits the 250 ms grace (`loop.go:118`) precisely so that a momentary probe never makes a dobby refuse.
Any process that probes the lock at that instant (`lyx batten overview` redrawing its dobby section, `lyx beam status`, the orch watcher, a `lyx beam start`, or the landing's own pass child) makes the new image believe a second dobby is live, and the re-exec ends the only dobby with exit code 0, silently, with an `already_running` envelope in the log as its only trace.
The overview and the watcher had both just re-exec'd themselves 7 s earlier and were drawing fresh frames, so probes were frequent in that window.
This is the pre-existing `runDobby` pre-check (also in 744e7842, `cli.go:203`), not something e6fbc14c introduced; e6fbc14c added a `Live` caller (`internal/loomcli/pawlwiring.go:131`, for beams) and more concurrent landing activity, but the pre-check race is older.

Confidence: high (about 85 percent) that the dobby ended through `runDobby`'s `already_running` exit right after its re-exec (last log line, `registerDobby` not reached, pid and state unchanged); low to medium (about 40 percent) that a momentary lock probe is what the pre-check saw held, since a probe holds the lock for microseconds and no holder can be shown on disk now.
No other holder is excluded: a second `lyx beam dobby` spawned in the exec window by some `EnsureLive` caller would produce the same line, and the log cannot tell the two apart.

### Ruled out

- A changed or refused argv: the dobby's command line, `SpawnDetached` and `selfexec` are byte-identical in 744e7842 and e6fbc14c, and a refused argv would print an error envelope, not `already_running`.
- A failed exec: `Check` logs at Warn and keeps running on a failed exec (`selfexec.go:77-81`); no Warn line is in `dobby.log` and the process is gone.
- The landing's Land or Worktree-Teardown rows, `runproc.End`, `EndSparing`, `EndRole`: teardown ends only the records of the slug owner `beam-start-and-ownership` (`ended=0`), the prime's dobby is registered under owner `_board` (`registerDobby`, `dobbydeps.go:60-66`), and `EndRole(..., RoleDobby)` is called only by `stopBeamDobby` (`internal/loomcli/pawlwiring.go:289`, a beam's slug) and `internal/orchcli/cli.go:263` (`lyx beam stop`, which ends the prime's `_board` dobby only when run for the prime, and no `argv=beam stop` trace exists in `.lyx/logs`); `EndSparing` is called only with a slug in `pairteardown/teardown.go:597` and a dobby owner in `fabriccli/anchordeps.go:243`.
- `lyx tender reap` / `tender` verbs: `runproc.End` in `tendercli/deps.go:113,321` runs only on an explicit tender verb for a named owner or slug, none ran at 15:17 (no trace file with argv `tender` in `.lyx/logs` between 15:17:37 and 15:18:24), and `tender status` only reports processes with a deleted exe.
- `update-plugins.sh`: it only runs `go run ./tools/deploy`, which mirrors the plugin files and builds `lyx`; it signals no process.
- A stop signal: SIGINT or SIGTERM would end `runDobby` through its context and print `{"stopped":true}`; the log has no such line.
- A hand-damaged state or a different pid: `state.json` still holds pid 96674 and the pre-deploy `start_time`.

### Not shown, to check on the fix

- Whether a SIGKILL or an OOM kill ended the dobby: no kernel or journal access, so only the exit path above is established.
- Which process held the lock at 15:17:47.88Z.

### Suggested fix direction (for the task that takes this note)

- Make the pre-check wait `lock.ProbeGrace` like `Run` does, or drop it and let `Run`'s own graced acquisition decide, so a momentary probe never ends a dobby.
- Let a process whose state record already carries its own pid (a re-exec, see `recordStart` in `loop.go`) retry the lock for longer than the grace, since the lock it needs was just released by its own exec.
- Log a refused re-exec start to `dobby.log` as a Warn naming the exit path, and post it to the orch, instead of printing only the envelope.
- Give the dobby a trace file, or have it log its re-exec at Warn, so the next such event leaves a line.
- Add the test the Expected section asks for: replace the binary under a running dobby while a probe loops on `dobbyengine.Live`.
