# Fix run window and shuttle bugs: launch line, status pane, loom fmt

Fixes seven defects noted on 2026-10-12 in shuttle, the status pane, the test kits, the edit directive and worktree teardown.

Closes #578

## What changes

- **Shuttle startup probe.**
  `start`'s probe in `shuttleengine` reads readiness from the run's events first: a session start followed by a turn start proves the provider is working, so a blank or collapsed pane no longer ends a live launch as died at the startup deadline.
  The pane capture stays for a launch the events do not yet prove ready, so a recognized trust or bypass gate is still dismissed.
  Wait's and attach's probe of a not-yet-started run keeps the capture rule, because its events file may hold an earlier incarnation's lines.
  An engine that does not record session starts keeps the capture-only probe.
- **Skill-load message on the launch line.**
  A spec that names skills now launches with the engine's skill-load message as the launch argument instead of an empty input box.
  Shuttle waits for that turn to end in the events file, then types the color, the retry turn for missing skills and the prompt pointer, each through the verified send.
  A load turn that times out skips its skills with a warning, and a pane that dies during it ends the startup died, as before.
  A spec without skills launches with the prompt pointer as before.
  `PATTERN-role-skills-typed` is reworded to "delivers it, on the launch line or typed", and `docs/overview.md` and the `roleskills_test.go` comment follow.
- **Status pane.**
  `statuspane` clears the screen and the scrollback history on every redraw, and a terminal resize wakes the loop at once on platforms with a resize signal.
  The loom checklist and batten's overview pane both use it.
- **`lyx loom fmt <file>...`.**
  Rewrites each named `.go` file inside the worktree to its `go/format` form, in-process.
  Every file is checked and formatted before any is written; a path outside the worktree (symlinks resolved), a non-`.go` path, a missing or unreadable file or one that does not parse refuses the whole call with nothing written, and the envelope names each refused file.
  Unchanged files are not rewritten or reported, and a rewritten file keeps its mode.
  The edit directive gains one line naming the verb, since it forbids every other rewriting tool.
- **Worktree teardown.**
  When `git worktree remove` exits nonzero having deregistered the worktree but left its directory refilled by a late writer, `fabricengine` finishes the removal with a bounded retry, at most two more attempts, with an injected sleep.
  Only `ENOTEMPTY` on Unix and `ERROR_DIR_NOT_EMPTY` or `ERROR_SHARING_VIOLATION` on Windows are retried; any other failure is returned as before.
  The leftover's removal appends a `KindPathRemoved` entry to `Remove`'s mutation record.
- **Stale-socket test helper.**
  The datagram-built stale socket moves from `reedengine`'s tests into `testkit/tmuxkit`, and `tmuxkit`'s dead-socket test uses it, dropping its fork-inheritance wait.
- **Push retry finding.**
  `gitrepo.PushWithRetry`'s doc comment records that `! [remote rejected] main -> main (failed)` is the remote refusing a ref update that raced another update of the records repository's `main`, and that the retry is the intended path.
  No behavior or log level changes.

## Design decisions

- The events proof is read only in `start`'s probe, whose events file is created fresh for the run, so only this launch's hooks can satisfy it. Signals are read through the engine's parsers, never by Claude hook name.
- The launch-carried load message skips the verified send's landing check; a turn end in the events file proves it instead, bounded by the skill-load timeout measured from the end of the startup probe.
- The leftover directory's ownership proof is that git ran after the gate passed the target, and a late writer is recognized by error value, never by git's stderr text.
- Clearing history on every redraw is deliberate; it discards a copy-mode position in a status pane.

## To check by hand

- The stacked checklist render may have another cause than earlier frames pulled back from history or left across a resize; confirm in a real run window that the render no longer stacks.
- The Windows-only held-open-file row for the worktree leftover retry skips on Linux and has not run; run the `fabricengine` integration suite on Windows.
- A run-window launcher that re-runs `lyx reed attach` in a loop would outlast the teardown retry; the removal then fails as before.
- `lyx loom fmt` unreadable-file refusal (`internal/loomcli/fmtverb.go`): no test row reaches it, for example a directory named `dir.go` beside a misformatted file in the same call. Check it refuses whole and names the file with its read error, and that any `EvalSymlinks` failure other than not-exist is refused with its own error rather than as "no such file".
- `removeWarpWorktreeDir` on Windows (`internal/fabricengine/remove.go`): the sharing-violation case, a file under the target held open by its writer such as a reed lock file, is covered only by the skipped row above.
- `cmd/lyx/tierpurity_test.go` gains an `allowedSpawners` entry for `internal/reedengine/socketfile_test.go`, since the shared helper spawns nothing; confirm that is the right place.
