# Card 11 — payload-shell-wait

**Edit:**
- `internal/shuttleengine/wait.go`
- `internal/shuttleengine/waitmark.go`
- `internal/shuttleengine/waiting_test.go`
- `internal/shuttleengine/hold_test.go`
- `internal/shuttleengine/run.go`
- `internal/shuttleengine/config.go`
- `internal/shuttleengine/template.yaml`
- `internal/shuttleengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Make a turn end waiting on live, payload-reported background work a wait, never a held stop (D8, #429).

In the wait loop:
- A background shell whose `Signal` is `SignalPayload` never expires: `expiredTurnEnd` keeps the turn waiting while such a shell is outstanding, bounded only by the run's own deadline and the liveness check, and it is never held and never notified.
  A shell reported by the transcript fallback, or one with no signal recorded, keeps today's `background_shell_wait_min` expiry, held stop and notice unchanged.
- When a payload-reported shell has been outstanding for `background_shell_wait_min`, `Wait` logs `shuttle: background shell past wait min` at Info once for that shell id, naming the run and the shell's label; the log changes no decision.
- In a gated run, a waiting turn end where every output file exists and every outstanding task is a payload-reported shell not matching `Spec.AwaitedShellPrefixes` finishes `OutcomeDone` at once (through the gated-boundary path, so the gate still runs): those shells are logged as today's waited-out shells are and added to `Result.ExpiredShells`.
  An outstanding fork or awaited-prefix shell keeps the gated run waiting as today.
- An ungated run with every output file present finishes Done at any turn end as today, whatever is outstanding.
- `shellWaitActive` keeps the `background shells` mark on while only non-awaited shells are outstanding and at least one is a payload-reported shell or a transcript-reported shell not yet waited out, so batten's quiet notice stays suppressed while such a shell runs; with a fork beside it no mark shows, as today.

What this lets through: a payload-reported shell that never ends now holds the run to its own Timeout (`master_timeout_min` for a Master), where it ends `OutcomeTimeout` as any timed-out run does; lyx reaps no shell.
That bound is stated in the `pollEventsTick` and `expiredTurnEnd` doc comments.

Tests, in `waiting_test.go` beside the existing shell-expiry tests (whose `oneShell` carries no signal and so keeps today's expiry):
- `TestPollEventsTick_ShellExpiry`'s table gains a task-list column and rows: a transcript-reported shell expires at the bound as today; a payload-reported shell is still waiting, not held, past the bound; a gated run with outputs present and only payload-reported unawaited shells finishes Done on the first tick, with the shells' labels in `ExpiredShells`, while the same with a fork or an awaited-prefix shell beside them keeps waiting; an ungated run with outputs present and a payload-reported shell finishes Done as today.
- `hold_test.go`: `TestWait_HeldTurnEndNotice`, whose `shellJump` rows already drive `Wait` past the shell bound to the deadline, gains a row whose outstanding shell carries `Signal: SignalPayload`: the run keeps waiting past the bound with no held turn end and no notice, and ends `OutcomeTimeout` at its deadline.
- One new top-level `Wait`-level function in `waiting_test.go`, since no existing test both jumps the clock past the bound and scripts a later turn end (`TestWait_HeldTurnEndNotice` takes either `shellJump` or `script`), and none captures logs: an autonomous run whose turn end lists a payload-reported shell logs `shuttle: background shell past wait min` once across several ticks past the bound, and finishes at the agent's next turn end.
- `TestShellWaitMark` gains a payload-reported row whose mark stays on past the bound.
- `cmd/lyx/testdata/test-budget.yaml`: the `internal/shuttleengine` row rises by 1, for the one new function.

Docs: `background_shell_wait_min`'s comment in `template.yaml` and `config.go` says it bounds transcript-reported shells only; `wait.go`'s file comment and `shuttleengine/doc.go` describe the payload-shell wait, the done-at-once rule and the Timeout bound.
`run.go`'s `Result.ExpiredShells` field comment, which today says each listed shell outlasted `background_shell_wait_min`, says the labels are the shells the wait stopped waiting on: transcript-reported shells it waited out past the bound, and payload-reported shells a gated run's turn end left behind at once.

**ImpactSummary:** Payload-reported background shells no longer expire into held stops; gated runs with outputs and only leftover unawaited payload shells finish at once; transcript-reported shells and ungated runs behave as before.
