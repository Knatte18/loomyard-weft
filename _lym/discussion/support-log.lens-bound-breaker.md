```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked the bounds, guarantees and claims about existing code in the decision record. I found two BLOCKING, seven MEDIUM and three LOW findings.

**BLOCKING — D10 with D6 and D9: the chase turns a deliberate hold into a severe page.**
- D6 and D9 have the unattended orch pause a run (open BLOCKING finding, or a would-be cancel) and post only a quiet `[operator] held` line.
- D10 ends the chase only on "a resume, a goto or an accept", and a pause is none of these.
- Counterexample: under `unattended` the orch pauses a run that holds an undeferred BLOCKING finding. The run stays halted with `settle: orch`. After `beam.halt_chase_min` it is chased, and one interval later D10 posts it as `[operator] severe halt`. D8 then mentions the operator and wakes the phone, which is the outcome D6, D8 and D9 promise not to produce.
- D10's bound "a run at a wait by design is never chased" does not cover an orch's pause.

**BLOCKING — D6: the arbiter can land a known BLOCKING defect unattended.**
- D6 lets the arbiter (lym) accept a budget round whenever every open finding is `deferred`. That includes BLOCKING findings and any autonomy level.
- The bound "the unattended orch lands no known open BLOCKING defect" covers only the orch.
- In `burler-step-fix.md`, a fixer defers a finding when "its only sound change leaves the surface". The code-review rubric (lines 119 and 127) has the fixer defer exactly the findings that need a parent ruling.
- So a BLOCKING finding deferred to the parent is accepted by the arbiter with no parent ruling, and lands under `unattended`. This contradicts D6's own rationale, which is to land no known defect unattended.

**MEDIUM — D5 with D4 and `push-retry-not-halt` D6: `resume` cannot act on a run in a long wait.**
- D5 waits whenever "its driver lock is held".
- D4's pause wait and the peer's network wait both hold the driver lock for as long as they last, possibly hours.
- Counterexample: the orch runs `lym loom resume` after `decision add` on a run in a network wait. The verb waits out the keep-warm cap and then refuses with `driver_not_parked`. The orch is back in the retry loop D5 was meant to end.
- D5 does not say that a waiting driver counts as parked, or that resume gets a distinct answer for it.

**MEDIUM — D7 and D10: the severe-post path and the program sender are unspecified, so routing breaks.**
- D10 says the dobby posts "by D7's path", but D7's path is the driver's hand-back, and the dobby is not the driver. The once-only settle-record guard has no counterpart in the dobby.
- D7 posts "as a program" without naming one. `outbound.go` (lines 218–219) routes `<shortname>:<NAME>` to the hub's orch channel, and only `<shortname>:<slug>:<NAME>` goes to a beam's orch.
- So the Open risks claim that posts "follow the run's recorded parent" is false unless the sender is a beam program.
- D12 checks the button of "the orch its severe posts would reach", which inherits the same ambiguity.

**MEDIUM — D7: "posts once per halt; the settle record holds that it posted" has no stated order.**
- If the driver posts and is killed before the record write, the restarted driver posts again.
- If it writes the record first and is killed, the halt is never posted.
- The record states neither order, nor which risk it accepts (at-least-once or at-most-once). The D7 acceptance criterion tests only a restart after a completed record.

**MEDIUM — D11: the `AwaitsReply` skip hides a stuck agent.**
- Per the exploration, `creelengine.Store.AwaitsReply` also answers true when the strand "holds an unread reply to its own message".
- An agent whose reply has arrived but who never woke is the stuck case the idle notice exists for.
- D11 skips it, and so does the "held" notice. The bound "an open question in creel" does not match what `AwaitsReply` answers.

**MEDIUM — D1: mixed builds read `unattended` as no autonomy.**
- A board `run_settings` or hub default of `unattended` read by an older binary is not a boolean. A stale driver awaiting hand-over, the dobby before redeploy, or another machine on an older build all read it that way.
- The exploration reports that `hubgeom.Autonomy` reads false on a failed read. Its halt notices would then stamp `autonomy: false`, and the orch would relay them to the operator as non-autonomous.
- The record claims "a board `run_settings` holding a boolean stays valid", but is silent on the reverse direction.
- D1's "any other value is refused" also does not hold on the `hubgeom` path, which degrades to false rather than refusing.

**MEDIUM — D3 and D4: the existing restart gap reopens D4's bounds.**
- `repairCount` counts re-step records by `StopKey`.
- `drive.go:809` restarts a key's repair count when a completed-wake stamp falls ≥ `wakeRestartGap` (10 min) after its last re-step.
- If the new D4 rows reuse that count, "at most twice per run at the row" resets after any 10-minute wake, and a slow-repeating launch failure gets endless fresh starts. The record is silent on the gap.
- Separately, D3 counts by row while settle records key by `StopKey`. Two different stop keys at one row each get their own bound under D4 but share one D3 count. The record never says which governs "per run at the row".

**MEDIUM — D2 table: rows contradict at the third stop.**
- The board-claim row gives orch "from the third stop" and severe at "third halt at the row", so both columns claim the same stop.
- The launch-failure rows give lym one stop, orch the second and severe the third, but the bounce-budget row reads "the second", then "on the third" in the study.
- Ambiguous verdicts at count three break D2's "one answer" guarantee and its tier0 test.

**LOW — D4: a pause wait may change hibernate's view of a pair.**
- In `hibernateengine/wake.go`, `WakeRun.Busy` marks an entry "whose pair a driver or a lock still holds".
- A driver in D4's pause wait holds its driver lock through a hub pause, which may mark the pair Busy across a hibernate.
- The record does not state that a pause wait yields to hibernate. I could not confirm the effect within my call budget.

**LOW — D8: the mention needs a configured operator.**
- The relay knows "the configured operator" only after `lym slack setup`. With no operator configured, a severe post carries no mention and nothing notifies the phone.
- D12 warns only about the button, not about a missing operator.

**LOW — D11: the Raddle-Write skip has a timing gap.**
- "A proposal that passes its checks" has no stated read time.
- A proposal that passed and was later edited, or whose agent died after passing, stays skipped. The record's claim that "a dead raddle agent still shows" holds only before the proposal passes.

Claims I attacked that held:
- D2's closed table and leaf package (stdlib only).
- D8's mention reaching only orch or program senders, since `outbound.go` drops other strands and other hubs.
- D9's narrowed severe classes are internally consistent.
- D12 is a notice-only change.
- D13's glossary lines.
- The Constraints entries for PATTERN-config-layers and PATTERN-agent-addressing.
- The Out scope against `push-retry-not-halt`.

Further tool calls:
1. `driverules.go` lines 230–260 and the `wakeRestartGap` grep: my context lacked how `repairCount` keys and restarts.
2. The keep-warm and hibernate grep: my context lacked where the keep-warm resolver lives and whether hibernate reads drivers.
3. The `hibernateengine` listing: my context lacked the hibernate package names.
4. `hibernateengine/wake.go`: my context lacked the meaning of `Busy`.
