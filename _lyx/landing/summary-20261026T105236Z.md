# Shuttle start retries a died launch and tmux EAGAIN; quieter hold notice

Fixes #870, #869 and #876.

## What changes

**reed: a tmux the host refused with EAGAIN is retried** (`internal/reedengine/overlay.go`).
Every tmux call goes through `TmuxCmd.run`, `runStdin` or `output`, and these now retry an exec that failed with `syscall.EAGAIN`: four tries in all, waiting 250 ms before the first retry and doubling each time, with each retry logged at Warn.
A fork or exec failure means tmux never ran, so a retry repeats no side effect; a tmux that ran and exited nonzero is returned at once.
Liveness checks, captures, sends and strand removals all inherit it, so the `fork/exec /usr/bin/tmux: resource temporarily unavailable` of #869 no longer reads as a dead pane or a failed removal.
`run` now delegates to `runStdin` with a nil stdin, so the retry lives in two places instead of three.

**shuttle: a died start launches again** (`internal/shuttleengine/run.go`).
`start`'s body past its checks moved into `Runner.launch`, one attempt from run-directory creation to a ready provider.
When an attempt ends `died` and the teardown removed its strand, `start` waits five seconds and launches again in a fresh run directory, up to `shuttle.yaml`'s new `start_retries` key (template `1`; `0` turns it off; negative is refused by `LoadConfig`).
A timed-out start, a start whose strand could not be removed, so two agents never run for one start, and a failure before the strand registered are never launched again.
Every failed attempt keeps its own run directory and `startup-capture.txt`, and the error returned is the last attempt's.

**shuttle: the startup teardown retries the strand removal and shows the head of the capture** (`internal/shuttleengine/wait.go`).
`abandonStartup` tries `RemoveStrand` three times, waiting one and then two seconds on the run clock, before its error tells the operator to remove the strand by hand.
The not-ready error now ends with `startupCaptureExcerpt`: the capture's last lines, preceded by its first lines when the capture is longer than one tail, with no line shown twice.
That keeps the first error line of a launch whose output scrolled away, which #870's tail alone hid.

**shuttle: no held notice for a turn end left waiting on background work** (`internal/shuttleengine/wait.go`, `hold.go`).
A held turn end built from a waiting turn end now records that it had background work outstanding.
That covers the gated run's plain-turn-end judgment on un-awaited shells and the ungated run's shell-wait expiry.
On an engine that meets `RequirementBackgroundWake`, such a turn end sends the parent no notice, defers none and leaves `NotifiedOffset` alone, because the session wakes when its own work completes.
The hold, the wait mark and the deadline are unchanged, and an Info line records the skip.
A held turn end with nothing outstanding is notified as before.

## Design decisions

- The launch line keeps its inline `--append-system-prompt` notice.
  Reed already writes the composed line to a per-strand launch script and types only the source statement, and `internal/shell` quotes embedded single quotes correctly, so the pane shell never parses the long notice as typed input.
  Claude Code documents `--append-system-prompt-file` for print mode only, so moving the notice to a file risks an interactive session that loses its deny notice.
  The two reported runs' captures were cleaned up with their run directories, so #870's root cause could not be read back.
  The start retry absorbs a recurrence, and the head of the capture now shows its first error line.
- The abandoned attempt still records `died`: the record's terminal outcomes are a closed set, and the attempt is torn down whatever caused it.
  An unverifiable liveness check no longer ends the start, because EAGAIN is retried before it surfaces and a failed attempt is launched again.
- The board entry asks that a turn answering a prompt the operator typed raise no notice either.
  Both reported cases had a background shell running, which the background rule covers.
  A turn the operator prompted with nothing outstanding is still notified, because shuttle holds no record that tells an operator's typed text from a parent's SendMessage answer, and silencing the parent's answer turns would hide a real follow-up question.
- The start retry is a `shuttle.yaml` key, not a constant, so a hub can turn it off or raise it, and hand-built test configs keep their single-launch behavior.

## Tests

- `TestTmuxExecRetriesEAGAIN` (`internal/reedengine`): EAGAIN then success, EAGAIN on every try and a nonzero exit, for both the discarding and the capturing exec, with the waits recorded through the injected pause.
- `TestStartup_DiedStartLaunchesAgain`: a died start launches again and returns the ready run in a second run directory; with the retry off, or with a strand the teardown could not remove, the start fails after one launch.
- `TestStartup_PaneNotLiveMidStartup`: new rows for a removal retried and then reported, and a removal that succeeds on a retry.
- `TestStartupCaptureExcerpt`: a short capture, one tail's worth, a little past one tail and a long capture.
- `TestWait_HeldTurnEndNotice`: the ungated shell expiry and the gated un-awaited-shell turn end send no notice on an engine that wakes on background work; the gated case still notifies on one that does not, and a plain stop still notifies.
- `TestLoadConfig_SubmitSettleKeys`: `start_retries` template default, zero and a refused negative.

`lyx gate test` tiers 0 and 1 over the diff are green.

## Docs

`internal/reedengine/doc.go` states the EAGAIN retry.
`internal/shuttleengine/doc.go` states the start retry, the teardown's removal retry and capture excerpt, and the background-work notice rule.
The template comment documents `start_retries`.
