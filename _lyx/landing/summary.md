# Shuttle start survives tmux EAGAIN and one died launch

A seat start no longer halts the run on one launch hiccup or a transient tmux `fork/exec` EAGAIN.
A failed start leaves no strand behind, and a turn that ends while the session's own background work runs no longer sends the parent a held notice.

Closes #870
Closes #869
Closes #876

## What changes

- `internal/reedengine`: every tmux call through `TmuxCmd` (`run`, `runStdin`, `output`) retries an exec that fails with `syscall.EAGAIN`.
  It makes a few attempts with a doubling delay, about two seconds in all, and each retry is logged.
  A tmux that ran and exited nonzero is never retried, and the last error returns unchanged.
  Liveness, capture, send and removal all inherit the retry, so an EAGAIN is no longer read as a dead provider.
- `internal/shuttleengine` start: a start whose attempt ended `died` and whose teardown removed the strand is launched again, in a fresh run directory, after a five-second wait.
  The new `shuttle.yaml` key `start_retries` (template default 1) counts the extra launches; 0 turns the retry off and a negative value is refused by `LoadConfig`.
  The returned error is the last attempt's, and each retry is logged at Warn with the failed attempt's run directory.
- `internal/shuttleengine` teardown: the strand removal is tried up to three times with a doubling run-clock wait before the error tells the operator to remove it by hand.
- `internal/shuttleengine` not-ready error: it carries the head of the last pane capture as well as its tail.
  A capture short enough for one tail is shown once, as before.
- `internal/shuttleengine` held notice: a held turn end built from a waiting turn end with background work outstanding sends no notice to the parent, on an engine that meets `RequirementBackgroundWake`.
  The turn end is still held and shown in the wait marker; the withheld notice is logged at Info.
  This covers both a gated plain waiting turn end and the ungated shell expiry.
- `doc.go` of both packages and the `start_retries` template comment describe the new behavior.
- `internal/loomcli` smoke fixtures that start a provider-less shuttle set `start_retries: 0`, so they still launch once.

## Design decisions

- The launch line keeps its inline system-prompt notice, so #870's suggestion to pass the prompt through a file is not taken.
  Reed already writes the composed launch line to a per-strand script and types only the source statement, and `internal/shell` quotes embedded single quotes, so the pane shell never parses the notice as typed text.
  Claude Code documents `--append-system-prompt-file` for print mode only, so a file could silently drop the deny notice from an interactive session.
  The cause of the reported `died` could not be read back, and the EAGAIN condition of #869 can explain it; the start retry and the EAGAIN retry cover it either way.
- An abandoned attempt still records `RunState.Outcome` `died`, because that terminal set stays closed.
  The EAGAIN retry keeps an unverifiable liveness check from surfacing as that outcome, and a failed attempt is retried.
- Only `died` with a removed strand is retried, so two agents never run for one start.
  A `timeout`, a teardown whose removal failed and a run whose output files exist are never retried.
- The EAGAIN retry sleeps through a `TmuxCmd` field a test replaces, not a package variable.
- #876's clause about turns that answer an operator-typed prompt is not implemented.
  Shuttle holds no record that tells an operator's typed text from a parent's `SendMessage` answer, and silencing the latter could hide a real follow-up question.
  A held turn end with nothing outstanding, including an operator-prompted one, is still notified.
  Both reported cases ran a background job, which the background rule covers.
- A provider that dies at once for a persistent reason now costs one extra launch and its startup window before the start fails.
  A host whose EAGAIN outlasts the tmux retry still surfaces it, and the shed's re-step remains the guard for a start that fails every attempt.

## Not run by hand

- No run against a live tmux under real fork pressure; the EAGAIN, start-retry and teardown paths are tested through injected sleeps and the fake clock.
