# Decision record — shuttle-start-hardening

## Goal

A shuttle seat start survives one launch hiccup and a transient tmux EAGAIN instead of halting the run, a failed start leaves no strand behind, and a turn that ends waiting on its own background work raises no held notice to the parent.

## Scope

- `internal/reedengine`: every tmux invocation through `TmuxCmd` retries a fork/exec failure with EAGAIN, with a short bounded backoff.
- `internal/shuttleengine` start path: a start that ends died and whose teardown removed the strand is retried as a fresh launch, a bounded number of times set by a new `shuttle.yaml` key.
- `internal/shuttleengine` startup teardown: the strand removal is retried a bounded number of times before the error tells the operator to remove it by hand.
- `internal/shuttleengine` startup teardown: the not-ready error carries the head of the last pane capture beside its tail.
- `internal/shuttleengine` held notice: a held turn end that came from a turn end with background work outstanding sends no notice to the parent on an engine that meets `RequirementBackgroundWake`.
- Docs: `internal/shuttleengine/doc.go`, `internal/reedengine/doc.go` and the template comment of the new key.

## Decisions

### Decision: the launch line keeps its inline system-prompt notice

Rationale: reed already writes the composed launch line to a per-strand launch script and types only the source statement, and `internal/shell` quotes embedded single quotes correctly, so the pane shell never parses the long notice as typed text.
A pane that shows the full line means the script write had failed and reed typed the line itself, or the line came from the provider's own output; neither is a quoting defect.
Claude Code documents `--append-system-prompt-file` for print mode only, so moving the notice to a file risks an interactive session that silently loses its deny notice.

### Decision: tmux fork/exec EAGAIN is retried at reed's one exec seam

Rationale: `TmuxCmd.run`, `runStdin` and `output` are the one place every reed tmux call execs, so liveness, capture, send and removal all inherit the retry.
A fork/exec failure means tmux never ran, so a retry repeats no side effect.
Bound: only an error whose chain holds `syscall.EAGAIN` from the exec itself is retried, a few attempts with a doubling delay totalling about two seconds; a tmux that ran and exited nonzero is never retried, and the last error is returned unchanged.
The sleep is a `TmuxCmd` field a test replaces, never a package variable.

### Decision: a died start whose strand was removed is retried as a fresh launch

Rationale: one launch hiccup, a provider that exits at once or a start-time send that fails on a host condition, should not halt the run, and the shed's own re-step runs only after the whole producer failed.
Bound: a new `shuttle.yaml` key `start_retries`, template default 1, counts the extra launches; a negative value is a config error and 0 turns the retry off.
Only an attempt that ended `died` with its strand removed is retried, so two agents never run for one start; a `timeout` start, a teardown whose removal failed and a run whose output files exist are never retried.
Each attempt gets its own run directory, so the failed attempt's capture stays findable, and the retry is logged at Warn with that run directory.
The returned error, when every attempt failed, is the last attempt's.

### Decision: the startup teardown retries the strand removal

Rationale: a removal that fails on a transient condition left an orphan strand and told the operator to remove it by hand.
Bound: three attempts on the run clock with a doubling delay from one second; the operator remedy stays in the error when the last attempt fails.

### Decision: the abandoned attempt still records `died`

Rationale: `RunState.Outcome`'s terminal set is closed at done, died, timeout and stopped, and the abandoned attempt is torn down whatever caused it.
What changes for an unverifiable liveness check is that EAGAIN is retried before it can surface and a failed attempt is retried, so it no longer ends the start.

### Decision: the not-ready error shows the head and the tail of the capture

Rationale: the tail alone showed only the end of a wrapped launch line and hid the first error line.
The error ends with the pane's first lines and then its last lines when the capture has more non-blank lines than one tail holds, and with the last lines alone otherwise, so a short capture is not printed twice.

### Decision: no held notice for a turn end left waiting on background work

Rationale: a session that ends its turn while its own background job runs wakes on that job's completion, so the parent has nothing to answer.
This covers both reported cases: Merriam's turn ends while its backgrounded verb ran, and the darn writer answering the operator while its backgrounded shell ran.
The turn end is still held, shown in the wait marker and logged; only the parent notice is withheld, and logged at Info instead.
Bound: it applies only to a held turn end built from a waiting turn end (the gated plain-turn-end judgment and the ungated shell expiry) and only on an engine that meets `RequirementBackgroundWake`; a held turn end with nothing outstanding is notified as before.
A turn the operator prompted with no background work outstanding is still notified, because shuttle holds no record that tells an operator's typed text from a parent's SendMessage answer, and silencing the parent's answer turns would hide a real follow-up question.

## Constraints

- `PATTERN-shuttle-provider-seam`: the background-wake check goes through the engine's `RequirementSupporter`, never a concrete engine; every negative startup verdict still consults `allOutputFilesExist` first, and the completion-signal scan's pinned sites are updated only where a site moves.
- `PATTERN-config-layers`: the new key is declared in `internal/shuttleengine/template.yaml` and classified in `ConfigSpec`, and `LoadConfig` refuses a negative value.
- `PATTERN-test-speed` and `PATTERN-test-isolation`: retries are driven by injected sleeps and the fake clock, never by real waits.
- `PATTERN-spawn-observability`: each retried tmux exec and each start retry is logged.
- `PATTERN-shell-guard-table`: no pane-shell string is built outside `internal/shell`.

## Auto-mode assumptions

- The `startup-capture.txt` and halt notes of the two reported runs were cleaned up with their run directories, so the cause of #870's `died` cannot be read back; the record treats it as a launch hiccup that #869's host condition can explain, which the start retry and the EAGAIN retry both cover.
- One extra launch is enough to absorb a hiccup; the shed's re-step and tackler still handle a start that fails every attempt.

## Open risks

- A provider that dies at once for a persistent reason now costs one extra launch and its startup window before the start fails.
- A host whose EAGAIN outlasts the tmux retry still surfaces it; the start retry and the shed's re-step are the remaining guards.

## Acceptance criteria

- A tmux exec that fails with EAGAIN and then succeeds returns the success; one that fails with EAGAIN on every attempt returns the error after the bounded attempts; a tmux that ran and exited nonzero is run once.
- A start whose first attempt dies before ready and whose strand is removed launches again in a fresh run directory and returns a ready run; with `start_retries: 0` it fails after one attempt as before.
- A died start whose strand removal fails is not retried.
- A startup teardown whose first strand removals fail and a later one succeeds reports the strand removed; one where every attempt fails keeps the by-hand remedy.
- A not-ready error for a capture longer than the tail carries the capture's first lines and its last lines; a short capture appears once.
- A held turn end from a gated waiting turn end on un-awaited shells, and one from the ungated shell expiry, send no parent notice on an engine meeting background wake, and still send one on an engine that does not.
- A plain held turn end with nothing outstanding still sends its notice.
- `start_retries` negative is refused by `LoadConfig`.
