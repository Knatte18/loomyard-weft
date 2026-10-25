```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. For each guard, refusal, branch and test row the cards add, I checked whether a real entry point can reach the state it guards. I found three defects.

1. **MEDIUM — card 5-capture-run / 9-drift-baseline-producer (D4, D3).** The row-started loom capture can fail its own precondition when it shouldn't.
   - `RunCapture` takes `CaptureLock` and only then runs `Check` (`CheckMainCapture`).
   - `StartCapture` returns as soon as `ProbeLock(CaptureLock)` reads held. The loom `Drift-Baseline` then commits and returns Done, and shed routes the run to `Plan-Write`.
   - If the child's `Check` reads the status after that write, the "current row is `Drift-Baseline`" check refuses a capture the row itself started. No record is written, and `Drift-Await` later halts on a missing record.
   - The same race opens on the D5 way forward: `lyx drift capture` followed by `lyx loom resume` moves the status off `blocked` at `Drift-Await` before the child's `Check` may run.
   - In short: the guarded state (current row has moved on) can occur after a legitimate start.
   - Fix: run `Check` before the lock reads as held, or have `StartCapture` wait for the child's fresh `running` record rather than the lock alone.
   - Evidence: card 5 Intent (`RunCapture` lock, then `Check`; `StartCapture` "returns the pid once `ProbeLock(CaptureLock)` reads held"); card 9 ("calls `StartCapture` and returns Done once the child holds the lock").

2. **MEDIUM — card 7-drift-verbs-cli (D4).** The `lyx drift capture` caller can never see the "capture already running" refusal.
   - The operator path runs `CheckMainCapture`, then `StartCapture` with `FallbackDetach`, which returns once `ProbeLock(CaptureLock)` reads held.
   - If another capture holds the lock, the new child refuses inside `RunCapture` and exits. `StartCapture` sees the other capture's lock as held and reports success.
   - So the `drift: a capture already holds &lt;lock&gt;` refusal never reaches the caller, and the "child exits first" error in card 5 is also skipped.
   - Fix: the verb should probe the lock and refuse first, or `StartCapture` should confirm the lock holder is its own child.
   - Evidence: card 7 Intent `capture` verb wiring; card 5 `StartCapture` and `RunCapture` refusal text.

3. **LOW — card 13-capture-teardown.** The Windows capture-job kill runs only while the task worktree is present.
   - A gone worktree skips `killCapture`, but `CaptureJobName` hashes only the worktree path, so it could still be computed.
   - A capture running in a pair whose worktree was removed by hand outlives teardown on Windows until its own timeout. The guarded site recurs on the gone path unguarded.
   - The card's own test row asserts the skip ("a gone worktree's does not"), so the skip is intended; the gap is unstated, not unseen.
   - Evidence: card 13 Intent ("`EndSession` calls it with the task worktree present"); `internal/pairteardown#Teardown.EndSession` gates the loop seize on `!gone`.

Every other guard and refusal the cards add can be reached:
- card 5 `CheckMainCapture`
- card 7 `RecordScope`, `Accept`, `Withdraw`, `Defer`, `Waive` and `CheckVerifyRow`
- the `Drift-Baseline` branches (card 9) and the `Drift-Await` branches (card 10)
- the `Drift-Verify` steps 1–8, including the resume detection (card 11)
- the activity reads (card 12)
- the entry gates (card 14). A routed write clears `EnteredRow`, so the gate is evaluated again when a reroute re-enters a Drift row.

Tool calls:
- `grep EnteredRow internal/shedengine/*.go`: to see whether a reroute re-entry evaluates a Drift row's entry gate again; my context lacked where `EnteredRow` is cleared.
- `sed -n 875,895p internal/shedengine/run.go`: to confirm a routed write clears `EnteredRow`; my context had only the grep line.
