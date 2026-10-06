# Card 15 — orch-start-never-attaches

**Edit:**
- `internal/orchcli/start.go`
- `internal/orchcli/start_test.go`
- `internal/orchcli/cli.go`
- `internal/orchcli/doc.go`
- `internal/orchengine/launch.go`
- `internal/orchengine/launch_test.go`
- `internal/orchengine/doc.go`
- `pattern/PATTERN-cli-cobra.md`

**Delete:**
- `internal/orchcli/handover.go`
- `internal/orchcli/handover_test.go`

**Intent:** D13, orch half: `lyx orch start` never attaches or switches a tmux client, for any caller, with or without `$TMUX`; attaching to reed is only `lyx reed attach`.

`launch.go`: `DecideStart`'s attach-only branch becomes a branch that changes nothing and reports the live strand and watcher.
Rename `StartAttachOnly` to `StartAlreadyRunning` with its comment saying strand and watcher are both live and start changes nothing.

`start.go`: after the start lock is released, the verb always prints the success envelope.
Delete the handover decision and call, the `actionAttachOnly` value becomes `actionAlreadyRunning = "already-running"`, and `startFields` drops its `attached` parameter and key; `hint` and `warning` stay.
`--no-attach` stays registered as an accepted no-op, hidden with `MarkHidden`, its usage text saying it is kept for existing callers.
Rewrite the verb's `Long` help: the strand-and-watcher-live case changes nothing, and the paragraph about attaching the terminal goes, naming `lyx reed attach` instead; update the file comment, which says start hands the terminal over.

Delete `handover.go` and `handover_test.go`.
The receiver's `reed` field in `cli.go` is read only by the handover, so drop the field and its assignment; the local reed engine still feeds the strand operations and `reedUp` as today.

Docs: `internal/orchengine/doc.go`'s module overview stops saying start hands the terminal over to reed's attach, and its sentence naming `DecideStart`'s outcomes ("attach only, spawn a watcher, or relaunch") says already running; `internal/orchcli/doc.go`'s worktree recipe drops `--no-attach` from its two `lyx loom start` steps; `pattern/PATTERN-cli-cobra.md`'s interactive-handoff exception list drops `lyx orch start`, beside the loom entries Card 14 drops.

Tests:

- `launch_test.go`: the `DecideStart` table's live-and-live row expects `StartAlreadyRunning`.
- `start_test.go`: the harness stops prepending `--no-attach`; `TestStart_LiveStrandAndWatcherDoesNothing` asserts the `already-running` action; `TestStart_NoStrandLaunchesAndSpawnsWatcher` drops its `attached` assertion; add one test, not parallel since it sets process-global `TMUX` with `t.Setenv` (named in a comment, per `PATTERN-test-speed`), running start with `TMUX` set, once with and once without `--no-attach`, asserting exit 0 and the success envelope in both runs.

**ImpactSummary:** Removes orch start's terminal handover and renames one StartAction; the start envelope loses its attached key, and the receiver drops its handover-only reed field.
