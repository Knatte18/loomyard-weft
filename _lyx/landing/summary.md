# Show what a run waits on and bring runs back up after a crash

`lyx loom status` and the reed status line now show the Go-side waits that looked like hangs, and a verify gate has a deadline.
After a crash or a halt, reed no longer revives finished strands, and `lyx batten run` brings a halted child's driver back.
The orch gets a resume verb that only wakes a parked driver.

Closes #408
Closes #409
Closes #420

## What changes

- **Waits in status.**
  The `waiting` field of `lyx loom status` reads `<producer>: <what> running <elapsed> (<detail>)`, for example `Webster-Burler: verify running 6m (go test -tags integration ./...)`.
  It covers every `verifytree.Verify` site: the Webster gate, the Webster-Burler round gate, the Publish and Finalize merge-in verify, and `webster verify`.
  Shuttle writes its own wait marker in the shuttle run directory for a gate entry and for the background-shell wait.
  The status hook reads the verify marker, then the shuttle marker of the worktree's live runs, then the parent-review note.
  A marker with a dead pid is skipped, and the search goes on.
- **Waits on screen.**
  Reed gains one op, `SetWaitMark`, that sets or clears the `@lyx_wait` and `@lyx_wait_start` pane options on a strand.
  Shuttle sets the mark on its agent strand for a gate evaluation or the background-shell wait, and clears it on every path out and at run start or attach.
  The landing merge-in verify sets it on the run's driver strand through a callback in `landingshed`'s deps.
  The default status line renders each marked pane as `<pane title> ⏳<label> <elapsed>` through a new `waits` token.
- **Verify timeout (#420).**
  `verifytree.Verify` takes a `time.Duration`, and production callers pass `verifytree.Timeout` (60 minutes).
  An expired timeout is a failed gate whose findings name the timeout, the log path and the log tail; it is not a returned error, and it writes no record.
  A caller's own cancel stays a returned error.
  On Unix `verifyrun` starts the shell in its own process group and kills the whole group on cancel or expiry.
  On Windows it kills the shell only and logs that.
- **reed (#408, #409).**
  `--unless-name` skips the add when the named strand exists, live or dormant.
  A strand can carry a done-when path list.
  `reed resume` drops a non-live strand whose listed paths all exist, and relaunches every other non-live strand.
  Shuttle fills the list with an ungated run's output files only.
- **batten.**
  The spawn confirmation is scoped to the batten process that wrote it.
  A restarted `lyx batten run` re-spawns a running child whose confirmation came from a dead process.
  For a halted or awaiting child with a dead driver strand it revives the pair's strands once per batten process, through reed resume, and leaves the run state and park marker alone.
  The halt reason names `lyx loom resume`, and also `lyx loom start` after a failed revive or when the child has no driver strand in reed state.
- **`lyx loom resume`.**
  A new verb, run in the task worktree, that types the resume line into a live, parked driver and removes the park marker, through the branch `lyx loom start` shares.
  It spawns nothing, attaches nothing and never reads the seed's driver.
  Every other state is a no-op success (done, or running with a live driver or a held run lock) or a refusal with a way forward, each with a row in `refusal-spec.md` and a test.
  The orch role stencil names it for every halted-child resume.

## Design decisions to check

- The wait mark is display only.
  No Go decision reads it, and `lyx loom status` reads the markers, never the pane, so a stale mark cannot hide a real wait.
  A failed mark op is logged and changes no verdict.
  `PATTERN-wait-mark-display-only` records this.
- The done-when list is the one place reed learns a strand finished.
  Only `shuttleengine` fills it, from an ungated run's `OutputFiles`, and only reed's resume reads it (`PATTERN-completion-signal`).
  A gated run's strand carries no list, because its outputs existing does not mean its gate passed, so it is relaunched as before.
- The verify timeout is a constant, not a config key.
- Process-group setup stays in per-OS files in `verifyrun`, using only the standard library (`PATTERN-shell-mechanics-seam`).
- Not built: a step-level watchdog goroutine from #420.
  The status note and the on-screen mark answer which gate a step is in.
- `lyx loom resume` tells the way forward for a halted run with no live driver by reed's strand table: dead, retiring or absent.
  A dead driver goes back through `lyx batten run` (or `lyx loom start` once a revive has failed or a batten run is already watching).

## To check by hand

- Open risk: the elapsed-time arithmetic in the status line (`%s` against a pane option inside `#{e|-:…}`) is verified only on the tmux in use here.
  If another tmux version does not expand it, the fallback is the start time instead of elapsed time.
- A crashed step leaves its wait mark on a surviving pane until the next set or clear on that strand.
  `lyx loom status` stays the truth.
- Watch a real verify gate and a background-shell wait with `lyx loom status` and the status line live.
- Run the Unix process-group kill against a verify command whose descendant holds the output pipe.
  The Windows shell-only kill was not run.
- `shuttle-turns` edits `internal/shuttleengine/wait.go` around the background-shell wait in parallel; compare the two changes.
- Run `go test ./...` and the `tmux`-tier tests.
