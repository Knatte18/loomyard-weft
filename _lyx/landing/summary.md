# Fix five small loop faults: review budget, idle clock, hold notice

Five independent faults in the loop machinery, each fixed in the module that owns it.

Closes #503
Closes #526
Closes #485

## What changes

- **Review budget (`internal/shedadapters`).**
  An approved review round no longer blocks with `bounce budget exhausted for <segment>-Burler` before the Bouncer judges it.
  The Burler round producer marks its two hand-backs for judgment, a completed round and an unjudged-round hand-back, as `BudgetExempt`.
  The Bouncer's `budgetReached` check at a CONTINUE verdict is now the one enforcement point of the budget on judged rounds.
  The Burler row's budget caps gate-failed rounds alone.
  The `roundBudgetExempt` special case and its decision-file read are gone, and the doc comments that described the old two-row offset are rewritten.
- **PR-Gate budget prose (`contracts/recipes`, `docs/overview.md`).**
  The loom and darn recipe comments and the overview's `lyx loom reject` sentence no longer promise five rejection rounds per task.
  They state the engine's episode rule: a Done by any row of segment `PR-Review` ends the gate's episode, so PR-Rework's Done resets the count and `max_bounces` never trips under the recipe.
  `max_bounces: 5` stays on both PR-Gate rows, because the status projection and the reject refusal read it.
- **Idle notices across suspend (`internal/battenshed`).**
  The quiet, no-live-run fallback and api-error idle clocks measure awake time, so a run is no longer reported idle for the hours the machine slept.
  A new injectable `Awake` clock sits beside `Now` in `deps` and defaults to Go's monotonic reading.
  Across consecutive probe checks the wait takes the excess of the wall delta over the awake delta, above a one-minute tolerance, as a sleep gap and subtracts the accumulated gaps from its idle durations.
  The notice text reports the awake duration.
- **Hold notice at skill load (`internal/shuttleengine`).**
  A turn end read before the start prompt's own turn start is logged at Warn and neither holds the run nor notifies the parent.
  `PromptOffset` is now the events-file size read immediately before the prompt is sent, after the load turns.
- **Held beside busy (`internal/shuttleengine`).**
  The held wait now ends at the first turn start past the held turn end, so a parent's answer that starts the agent returns the loop to `running` instead of leaving `held` beside a session state of `busy`.
  The `session state disagrees` Warn line gains a field saying the loop's classification stands, and the package doc says so.

## Design decisions

- Unbounded cycling through the exempt hand-backs is ruled out by the Bouncer's counting.
  Every Bouncer Stuck is counted except the one `settleUnconverged` returns for a recorded budget continue, so a judge that never recovers halts within `review_max_bounces` counted Bouncer Stucks.
  A gate-failed round after a budget continue is bounded by the Burler row's counted gate-failed Stuck.
- The sleep-gap wall delta is taken with Go's monotonic part stripped.
  `Time.Sub` over two `time.Now` readings returns the monotonic difference, which is the awake delta itself and would never show a gap.
- A gap seen in a check whose activity stamp changed is dropped rather than added, since it may lie before the new activity.
  The gap is only ever subtracted, so the fix can delay a notice and never raise one earlier.
- On a platform whose monotonic clock counts suspend, no gap is observed and batten behaves as before.
- The turn-start reading for the hold guard and the held-wait end goes through the provider-neutral `SessionSignalParser` capability, from the bytes the tick parses, and never from the shadow's fold.
  The guard applies only when the events file holds a turn start before `PromptOffset`.
  An engine without the capability, a zero `PromptOffset` and a file with no earlier turn start keep the old behaviour.
- Ending the held wait early changes no verdict, notice or return.
  The reject verb's budget refusal and its message are unchanged.

## To check by hand

- A provider that writes a turn start for the load turns but none for the prompt turn would have every turn end skipped with a Warn line, and the run would wait out its deadline with no hold notice.
  Look for the Warn lines in a real run of each provider.
- The one-minute sleep-gap tolerance is a guess: a machine that stalls for over a minute without sleeping, such as a frozen VM, also has the stall subtracted and a notice delayed by it.
- Suspend behaviour was tested only with fake clocks; a real suspend and wake on Linux has not been run.
- `cli-surface`'s `held` beside `busy` line was assumed to follow a parent's answer, and the original #485 stray turn end is no longer on disk; both are fixed by cause and class, not reproduced.
- The reject verb's budget refusal is now documented as unreachable under the recipe; a future per-task budget would reopen it.
- The first verify run failed on `envkit`, `battencli`, `loomcli` and `loomrecipe` tests, fixed in `e4fcc3847` by listing `InnerRun.Awake` as nil-legal in `envkit` and asserting that every Burler hand-back is exempt in the `loomrecipe` escalation test.
  Check those two edits.
