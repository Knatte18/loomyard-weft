# Decision record — creel-stage1-fixes

## Goal

Fix the eight creel defects of the 2026-10-16 creel stage 1 re-test (#677, #678, #681, #682, #683, #684, #685, #686), #677 first, so each re-test observation meets the expectation its board section states.

## Scope

In:

- `internal/creelengine`: journal tail repair (#677), stall freshness (#682), the client's connect wait (#683), carried records in `log` (#684), the removed-holder sweep timing (#685), the no-input-box stall and TTL-capped probe waits (#686).
- `internal/creelcli`: the stderr fallback for an unwritable stdout (#681), the spawn seam's exit signal (#683), the Idle adapter's new stall (#686), and the llm smoke test (#678).
- `internal/shuttleengine`: `IdleProbe` reports a pane that shows no input box (#686), and an engine-neutral seam that accepts a provider's startup gate on a live strand (#678).
- Docs in the same commit: `internal/creelengine/doc.go`, the creel worker and sweep lines of `pattern/PATTERN-daemon-wakeups.md`, and the C4, C6, C8, C10 and C11 watch lines of `tools/sandbox/SANDBOX-CREEL-SUITE.md` where their timing or outcome text changes.

Out:

- The reed defects of the same re-test, #679 and #680.
- C13 and every other part of `SANDBOX-CREEL-SUITE.md` that describes strand placement: C13's stacked half is the board note `creel-c13-stacked-rewrite`, and the placement sweep is the running task `reed-add-parentless-placement`.
- Creel stages 2 and 3 (`creel-notices`, `creel-agents`).
- Any change to `internal/output` or to other modules' handling of an unwritable stdout.

## Decisions

### D1 (#677): a torn last line is cut whether or not it ends in a newline

Decision: the tail repair at journal open treats the last non-blank line of the newest day file as the tail.
When that line is not valid JSON, it truncates the file to the end of the line before it, newline included, and logs the existing Warn with the byte count.
A valid last line missing its newline still gets the newline appended.
The fold's skip rule and the repair's cut rule use the same definition of the last line, so a line the fold skipped is always the line the repair cuts.
A malformed line anywhere else still fails the start with its way forward.
Rationale: the fold already skips a torn last line that ends in a newline, but the repair only cuts a tail with no newline, so the skipped line survived and became a middle line on the next append.
Bound: only the last non-blank line of the newest file can be cut, the same line the fold already skips, so no record a fold would apply is ever removed.

### D2 (#681): a creel refusal whose stdout write fails goes to stderr

Decision: creelcli's refusal path writes the JSON error envelope to stdout with a checked write.
When that write fails, it writes one line `creel: <error>` to stderr, and the exit code stays 1.
For `read`, the error is the failed write of the messages, so the stderr line names it and says the messages stay unread.
A stdout that is a closed pipe ends the process by SIGPIPE before any line; the mail still stays unread, since no acknowledgement is sent.
Rationale: a failed stdout write cannot carry its own error envelope, and stderr is the only channel left.
Applies to every creel verb's refusal.

### D3 (#682): a stall reaches a reader only from a fresh reading

Decision: a receiver's standing stall records, beside the time the stall began, the time a probe last read it.
A new `inject --wait` is seeded with the standing stall only when that last reading is fresh, no older than twice the probe backoff ceiling.
An expired outcome, its `expired` record's reason and its answer name the stall only under the same freshness rule.
A stale stall is dropped.
When a probe reads a stall a waiter has not been told since its last change, that waiter is told, so a waiter seeded with no stall still hears a stall that persists; a waiter hears each change it sees once, so A, then B, then A again reaches it three times.
The stall's printed time stays the time it began.
Rationale: the standing stall is cleared only by a probe that reads the pane, and a probe skips the read while a hold keeps a receiver's injects back or nothing is due, so a cleared draft stayed standing and a later waiter was told it.

### D4 (#683): the client stops waiting once the daemon it started has exited

Decision: the spawn seam the client is given answers, beside the pid, a signal that fires when that process exits; creelcli reaps the detached child in the background to fire it, and logs the exit with pid and status at Info.
The signal belongs to one spawn: a reconnect that spawns again waits on its own spawn's signal.
Between connects the client waits on the backoff timer or that signal, whichever comes first.
On the signal it makes one last connect, since a spawned daemon that lost the start race to another daemon exits while that other daemon answers; when that connect fails it returns the existing error that names the socket and quotes the daemon log.
Once a connect has succeeded, the signal is no longer read, so a daemon that dies later affects the call only through its dropped connection, as today.
A spawn seam that gives no signal keeps today's backoff.
Rationale: today the client sleeps its whole connect backoff, about 23 s, before it reads the log of a daemon that died at once.
Bound: the signal only shortens the wait; the attempt cap and the backoff ceiling stay as they are.

### D5 (#684): `log` shows a carried entry as a `carried` event

Decision: a carried record adds one entry to the pool's event list, with event `carried`, the snapshot's id, sender, receiver and kind, and the carried record's own time.
This holds both when a fold applies a carried record and when retention writes one into the running daemon's journal, so `log --id` finds it in either case.
A message carried again at a later retention gets one `carried` entry per carry.
The entry's other filters (`--from`, `--to`, `--since`) match it like any other event.
Rationale: a carried message is live state the receiver's `inbox` counts, and `log` is the audit view of it; the carried entry stands in for the original events retention deleted.

### D6 (#685): the removed-holder sweep confirms a missing holder fast and backs off less

Decision: two changes to the sweep, keeping its two-misses-in-a-row rule and its rule that a failed listing counts toward no streak.

- After a sweep that misses a holder for the first time, the next sweep runs after a confirm interval of 10 seconds instead of the backoff wait.
- The sweep's backoff ceiling drops from 4 minutes to 50 seconds, with the 30-second base unchanged.

Rationale: a removed holder is then settled within the ceiling plus the confirm interval, 60 s, for every holder the sweep covers, an orch-role holder included, which has no delivery worker.
Bound: the confirm interval and the ceiling stay above the one-second floor, and the sweep still runs only while unread mail or a queued inject exists, so it stays within PATTERN-daemon-wakeups.
A holder is released only after two successful listings at least 10 s apart both miss it.

### D7 (#686): a pane with no input box names a stall

Decision: `shuttleengine.IdleProbe` gains a field reporting that the pane shows no input box to an engine that reads one, taken from the existing `InputBoxReader` reading on the pane-probe path and from the existing no-input-box readiness reason on the readiness path, with no new provider-specific check.
creelengine adds the stall `no-input-box`, whose reason says the pane shows no input box: it runs no agent, or a dialog covers the box, and the delivery waits until the box shows.
The creel adapter maps the field to that stall, after too-short and draft in precedence.
Rationale: a bash strand or a pane under a dialog never reads idle and today names no reason, so an expired inject into it says nothing.
Bound: the stall only names a reason; it changes no idle answer and types nothing.

### D8 (#686): an inject expires at its TTL, not at the next backed-off probe

Decision: while a receiver is live and unpaused, its worker's wait before the next probe is capped at the remaining TTL of its soonest-expiring queued inject, so the probe that ages it runs when the TTL runs out.
The cap never goes below the one-second floor.
Rationale: expiry is checked only at a probe, and the doubling backoff overshoots the TTL; both observed overruns, too-short and never-idle alike, match the backoff's probe times.
Bound: the cap only shortens a wait, and only while an inject is queued and its TTL runs.

### D9 (#678): the llm smoke test prints its pane, places its strand in its own window and gets past the folder-trust gate

Decision: the llm smoke test's idle wait prints the strand's last pane capture when it gives up.
The smoke helper's `reed add` passes `--anchor own-window`, the #673 workaround CLAUDE.md requires for every `reed add` that names no `--parent`, so the strand is not split into the selvage shell window and drawn too short.
Before the idle wait, the test waits, under `smokePaneWindow`, for the claude pane to show either its input box or a startup gate; on a gate it accepts it through an engine-neutral `shuttleengine` seam that runs the provider's own startup-gate classification and accept sequence on a live strand, reached through `shuttleengines` like the test's other shuttle calls.
The strand stays a hand-added `lyx reed add --cmd claude` strand, since a hand-added strand is what the test proves.
Rationale: two causes fit the never-idle failure: the strand split into the shell window, where the pane can be too short for the box, and the folder-trust gate a fresh temp hub shows, which no shuttle startup dismisses for a hand-added strand; printing the capture makes any third cause readable.
Bound: the new seam plays only the provider's own gate-accept sequence, only on a strand the caller names, and only when the provider's classification reads a gate.
Accepting the trust gate adds the temp hub's path to the operator's Claude trust list, since llmkit does not isolate Claude's config; that is accepted for an llm run the operator starts by hand.

## Constraints

- PATTERN-creel-store: only `internal/creelengine` writes the journal and reaches a pane, through its `Delivery` seam; D7's new probe field is read by creelcli's adapter, never by creelengine directly.
- PATTERN-shuttle-provider-seam: the trust-gate needles and accept keys stay under `internal/shuttleengine/claudeengine`, and creelcli reaches the D9 gate seam only through `shuttleengine` and `shuttleengines`, never by importing `claudeengine`.
- PATTERN-daemon-wakeups: D4, D6 and D8 change waits in listed unbounded-wait files; every periodic wake stays at least one second apart, no test waits out an interval, and the background file's creel worker and sweep lines are updated in the same commit.
- PATTERN-spawn-observability: D4's background reap waits on the spawned daemon, so its exit is logged.
- PATTERN-tolerant-records: the journal gains no new record shape; D5 reads carried records as they are.
- PATTERN-test-speed and PATTERN-test-economy: engine changes are tested at tier 1 with the injected clock; a test extends an existing table where one covers the behavior.
- PATTERN-test-speed's llm rule: no gate runs the `llm` tier; D9 is compiled by `go vet -tags llm` and run by the operator.
- The `test-budget.yaml` rule in CLAUDE.md: a card whose new top-level tests push a package past its row lists that file under `Edit:`.

## Auto-mode assumptions

- #681 is fixed for every creel refusal, a widening of the board's `read` example.
- #682 drops a stale stall rather than marking it as past; the board allows either.
- #686's first half names a reason rather than only documenting the outcome; the board allows either.
- #678's cause is taken to be the strand placement or the folder-trust gate, both addressed.
- D6's 10 s confirm interval and 50 s ceiling are picked to meet C11's "about 60 s".

## Open risks

- #678's pass is proven only by the operator's by-hand run of `lyx gate test --tags llm ./internal/creelcli`, since no gate runs the llm tier; if the capture shows a third cause, that run names it.
- D7's no-input-box stall also fires while a claude pane boots or a dialog is up; each change journals one `stalled` record.
- D6 shortens the two-miss guard from at least 30 s to at least 10 s: a strand absent from reed's persisted state for longer than that while it is re-added under the same name loses its held mail back along its path.
- The running task `reed-add-parentless-placement` edits `SANDBOX-CREEL-SUITE.md` and may touch reed-add calls in test helpers; this task keeps its suite edits to C4, C6, C8, C10 and C11, and whichever lands second merges the other's lines.
  Once #673 is fixed, the `--anchor own-window` in the smoke helper is removed with the other workarounds by that fix.
- A sandbox re-check from inside a strand pane runs the production `lyx` until #680 is fixed, so a dev build's daemon is checked from the shell, not from a strand.

## Acceptance criteria

- A newest day file whose last line is torn and ends in a newline is cut at journal open; the next append follows the last valid line, and a restart folds with no error.
- A torn last line followed by blank lines is cut the same way, and a newest file whose only line is torn is cut to empty and gets its high-water record on the next append.
- A malformed line that is not the newest file's last non-blank line still fails the start with its way forward.
- `lyx creel read` with an unwritable stdout exits 1, writes a `creel:` line to stderr, and leaves the mail unread; a following `read` returns it.
- An `inject --wait` started after a stall was cleared, while the receiver's probes skip the pane (a hold), prints no stall line and ends injected.
- A stall that persists is told to a waiter that arrived after it began, once; a stall that changes A, B, A is told three times.
- An expired inject names no stall that was last read before the freshness bound.
- A verb whose spawned daemon exits at once returns its daemon-log error without waiting out the connect backoff, on the injected clock, and the exit is logged.
- A spawned daemon that exits because another daemon already serves still leads to a successful connect.
- A spawn seam with no exit signal keeps today's connect backoff.
- `log --id <n>` lists a `carried` entry for a carried message, both after a fold of a journal holding the carried record and after retention carries it in a running daemon.
- A removed holder of unread mail, an orch-role holder included, has its mail returned or undelivered within 60 s of the removal on the injected clock, whatever the sweep's backoff had grown to.
- A holder missing from one listing and present in the next keeps its mail, and a failed listing counts as no miss.
- An inject into a live pane that shows no input box expires with stall `no-input-box` in its answer and its `expired` record.
- An inject whose receiver never reads idle expires within one second of its TTL on the injected clock, for TTLs that fall between backoff steps.
- The llm smoke test compiles under `go vet -tags llm`, adds its strand with `--anchor own-window`, and on an idle-wait failure prints the pane capture.

## Notes for the plan writer

- Put #677 in the first card.
- #677: `repairTail` and `foldJournal` must share the last-line definition.
- #683: under go test `spawnDaemon` spawns nothing, so the client tests drive the exit signal through a fake seam.
- #678: `claudeengine/startup.go` exports no gate function today, and the engine seam that shuttle's start uses for gates is the place to extend for D9's seam.
