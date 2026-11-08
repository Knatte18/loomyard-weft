# A test binary pinned as a tmux job re-runs its own suite and fork-bombs the machine

`test-binary-fork-guard` · Note · bug, reed, gate, testkit

## Seen (orch, 2026-11-08)

Neve froze twice on 2026-11-08, at 14:37 and 15:40, and needed a hard restart each time.
The journal's memory samples show a fork bomb of `reedengine.test`, `tmux` and `bash` processes: from 125 to 537 test processes in four minutes, load above 800, memory and swap full.
The cause was in the running task `slack-presence-auto` (commit `4aa372a7`): a reed status job built from the running binary's own path (`composeSlackStateCommand`, `internal/reedengine/panebin.go`).
Under `go test` that path is the test binary, so each tmux session a test made ran the test binary again, which ran the suite and made more sessions.
That run carries a parent decision to fix its own code.

## Idea

Guard against the whole class, not only this job:

- Every package whose code pins a command built from `os.Executable` into tmux (status jobs, click bindings, hooks) gets a `TestMain` that exits at once when the test binary starts with arguments that are not test flags.
- Or one shared helper resolves "this binary" and refuses to return a test binary's path, so production code can never pin one.
- The gate could also cap the processes a test run may spawn, so a loop fails the gate instead of freezing the machine.
