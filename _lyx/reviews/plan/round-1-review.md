---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/05-hold-after-prompt-turn-start.md (Intent, The guard); _lyx/plan/06-held-wait-ends-at-turn-start.md (Intent)
    summary: The turn-start read sits after the batch's events are parsed and only when the last is neither waiting nor done, so a turn start arriving alone in a tick yields no event, returns early and never lifts the guard, and every later turn end of the run is skipped
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/plan/05-hold-after-prompt-turn-start.md (Intent, The guard)
    summary: The position test that skips a turn end followed in the batch by a turn start is not conditioned on the guard being armed, so it skips turn ends after the guard is lifted and on runs that never armed it, against D4's Bound and AC11
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/06-held-wait-ends-at-turn-start.md (Intent)
    summary: turnStartRead only fires while a held wait is already on show, so a held turn end and the turn start after it read in one batch begin a hold that lasts the whole following turn, the held-beside-busy pair D5 removes
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/04-prompt-offset-before-send.md (Intent, test paragraph)
    summary: The new TestStartup_SkillLoadTurns row is to append a line through the fake clock's step hook, but skillStartFixtureWith builds a plain fakeClock with no step hook and is not a card target
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/03-batten-awake-idle-clock.md (Intent, Helpers in notice.go)
    summary: observeClocks's first-observation branch records the readings but does not say it records the stamp, and an unrecorded stamp reads as changed at the second check, dropping the first gap and failing AC7
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/02-pr-gate-budget-prose.md (Intent, grep paragraph)
    summary: Card 2 says the remaining never-resets Go hits describe the Burler row's episode and stay true, but internal/shedengine/progress.go's Routing.Bounces doc says the Burler episode never resets, which D1 and episodeStuckCount's own doc contradict
---

### [BLOCKING] The guard is never lifted when the prompt's turn start arrives in its own tick

**Location:** `_lyx/plan/05-hold-after-prompt-turn-start.md`, Intent paragraph "In `pollEventsTick`, once the batch's events are parsed and the last one is neither waiting nor done, and before the held turn end is returned: compute the batch's turn-start offsets…"; `_lyx/plan/06-held-wait-ends-at-turn-start.md`, "sets it from the batch's `turnStartOffsets`, the same read card 5 makes".

**Issue:** `pollEventsTick` (`internal/shuttleengine/wait.go:621`) returns through `expiredTurnEnd` as soon as `len(events) == 0`.
A turn-start line yields no event: `claudeengine/signals.go:145` returns the turn-start signal with `Event` false, the decision record states "the events parser yields no event on a turn start", and both `fakeEngine.ParseEvents` and `sessionFakeEngine`'s inherited parser skip `START`.
In a real run the prompt's `UserPromptSubmit` line is written when the turn begins and the `Stop` line when it ends, normally many poll intervals apart.
The tick that reads the turn start alone therefore returns before card 5's read, `run.offset` moves past the line, and the guard stays armed.
The next tick reads the turn end alone, finds no turn start in its batch, and skips it with a Warn.
That happens for every turn end of the run, so a provider that does write the prompt's turn start falls into the failure mode D4's Bound reserves for a provider that does not ("every turn end of the run is then skipped… waits out its deadline with no hold notice").
D4 requires that "once a turn start is read the guard is lifted for the rest of the Wait".
A batch whose last event is `EventWaiting` has the same hole, since card 5 reads turn starts only when the last event is neither waiting nor done.
Card 6 inherits this: it reads "the same read card 5 makes", which contradicts its own claim that "a turn start that yields no event still ends the hold", the D5 case.
None of card 5's test rows catches it: the AC10 row appends "the `START` plus `STOP:question` that follow" with no requirement that they land in separate steps, and the one-batch row puts them together on purpose.

**Fix:** In card 5, compute `turnStartOffsets` over `data` right after `ParseEvents` succeeds and `run.offset` advances, before the `len(events) == 0` return, and lift the guard on any turn start there, whatever the batch's last event is.
State that card 6's `turnStartRead` is set from that same early read.
Add to card 5's `TestRun_Wait_HeldTurnEnd` rows one in which the prompt's `START` is appended in one step and `STOP:question` in a later step, asserting the second step holds.

### [BLOCKING] The same-batch position test is applied whether or not the guard is armed

**Location:** `_lyx/plan/05-hold-after-prompt-turn-start.md`, Intent: "With the guard still armed, or with every turn start of the batch positioned after the held turn end's own line…, the turn end is not a task turn end".

**Issue:** The second disjunct is not limited to a guard that was armed when the batch was read.
Taken literally, once the guard is lifted, or on a run that never armed it, a batch holding a turn end followed by a turn start skips the turn end: no hold, no notice.
That happens whenever an agent ends a turn and the parent's answer starts the next within one poll interval.
The cases AC11 says keep today's behaviour also reach it: a `sessionFakeEngine` run with a zero `PromptOffset`, or with no `START` before `PromptOffset`, still gets non-nil offsets from `turnStartOffsets`, since the capability is present.
D4's Bound says "After that start every turn end counts as today", and AC11 requires those runs to hold the first turn end as today.

**Fix:** Reword the rule: a turn end is skipped only when the guard is armed at the start of the batch and no turn start in the batch precedes the turn end's own line.
A turn start anywhere in the batch still lifts the guard for later batches.
Add an AC11-style row with `sessionFakeEngine`, a zero `PromptOffset`, and a `STOP:question` followed by `START` in one step, asserting the hold notice still goes.

### [MEDIUM] A turn start read in the same batch as the held turn end never ends the hold

**Location:** `_lyx/plan/06-held-wait-ends-at-turn-start.md`, Intent: "`turnStartRead`, true when the latest `pollEventsTick` read a turn start positioned past `heldOffset` while a held wait was on show".

**Issue:** In `Wait` (`wait.go:250`–`271`) the hold is begun after the tick's read.
When one batch holds `STOP:question` and then the `START` of the parent's answer, no held wait is on show at read time, so `turnStartRead` stays false.
`beginHeldWait` then shows `held`, and no later batch carries a turn start past `heldOffset` until the next turn begins.
The loop reads `held` beside a session state of `busy` for the whole answered turn: the pair D5 exists to remove ("the held wait ends at the first turn-start signal read past the held turn end's offset").
Once F2 is fixed this batch shape reaches the hold path on every run, whatever state the guard is in.

**Fix:** State in card 6 that when a tick returns a held turn end and the same batch holds a turn start past `held.offset`, the held wait is not shown, or is ended in that tick.
The hold notice still goes, as D5's Bound requires.
Add a `TestRun_Wait_HeldTurnEnd` row that appends `STOP:question` and `START` in one step, asserting the hold notice is sent and the marker reads not held at the next step.

### [MEDIUM] Card 4's test names a fixture hook that does not exist

**Location:** `_lyx/plan/04-prompt-offset-before-send.md`, test paragraph: "a line is appended to the events file after the last load turn end and before the prompt goes out, through the fake clock's step hook".

**Issue:** `TestStartup_SkillLoadTurns` (`startup_test.go:913`) runs through `skillStartFixture` → `skillStartFixtureWith` (`startup_test.go:855`), which builds `newFakeClock(time.Now())`.
That plain `fakeClock` (`wait_test.go:28`) has no step hook.
The step-hook clock is `multiStepClock`, which only `TestRun_Wait_HeldTurnEnd` builds.
`skillReed` (`fakes_test.go:460`) has no hook either, so the row cannot be written without changing a fixture member that card 4 does not list.

**Fix:** Add the fixture member the row needs to card 4's `**Edit:**`: `internal/shuttleengine#skillStartFixtureWith` to take a clock or hook, or `internal/shuttleengine#skillReed` / `skillReed.SendText` to append a line after the last load turn's `STOP:loaded`.
Name that mechanism in the Intent instead of "the fake clock's step hook".

### [LOW] `observeClocks` does not say the first observation records the stamp

**Location:** `_lyx/plan/03-batten-awake-idle-clock.md`, "`observeClocks` is the one updater: with no previous readings it records them and returns".

**Issue:** "them" reads as the wall and awake readings.
If the stamp is not recorded on that first call, the second call sees a known stamp that differs from the recorded zero value.
It then drops the accumulation without adding the gap observed between the two checks.
AC7's own scenario, hours of wall time across one probe check after the first, would then raise the quiet notice on wall time.
The planned AC7 subtest would catch this, but the Intent as written prescribes the failing order.

**Fix:** State that the first observation records the readings and the stamp, when known, so only a stamp that moves after a recorded one drops the accumulation.

### [LOW] Card 2's grep conclusion misreads `Routing.Bounces`'s doc comment

**Location:** `_lyx/plan/02-pr-gate-budget-prose.md`: "the Go comments the first phrase hits are rewritten by card 1 or describe the Burler row's own episode and stay true".

**Issue:** `internal/shedengine/progress.go:102` reads "a Burler returns Stuck every round and never Done, so its own episode never resets and is never counted".
`episodeStuckCount`'s own doc (`internal/shedengine/run.go:505`) says a Burler-round row's episode ends when its segment's Bouncer passes.
D1 says the Burler row's "episode ends at the segment's Done or a goto into it".
So that comment is not true, and card 2's claim that it is misleads the implementer.
D1's comment list names only the `shedadapters` sites, and its notes say `progress.go`'s `Bounces` needs no change.
That bars a code change, not a comment correction, so this is a wording defect in the plan, not a missing D1 item.

**Fix:** Correct card 2's sentence so it does not call the `progress.go` comment true.
Either leave that comment out of scope explicitly, or let card 1 correct the clause on the Burler episode in `internal/shedengine#Routing.Bounces`'s doc comment, which D1's rewrite-to-the-new-rule direction covers.

## Settled focus questions with no finding

- Card order between cards 5 and 6: card 6 depends on card 5's `turnStartOffsets` and on card 5's per-row engine choice and per-step held expectation in `TestRun_Wait_HeldTurnEnd`, both from the earlier card. It depends on nothing a later card does. The shared test rows do not collide: card 5's `TestWait_LogsSessionStateBesideItsClassification` row asserts the skip Warn and the classification, and card 6 adds a separate row and the `stands=loop` assertion on the existing disagreement row.
- Card 3's glyphs: `InnerRunDeps`, `NewInnerRun`, `childWait` and `innerRunProducer.judgeAgents` cover every existing symbol the Intent changes, and the new helpers are declared under `**Create:**`. The `Awake` seam's bound is stated: the platform bound, the one-minute tolerance, and that the subtraction can only delay a notice. No wake is added.
- Card 5's guard bound: the Intent states it in the R3 form D4 gives. F1 shows that the bound as written is not what the planned code delivers. The card-5 rows do not duplicate existing `TestRun_Wait_HeldTurnEnd` rows: the existing rows all use `fakeEngine` with a zero `PromptOffset`, and the new AC11 rows isolate the capability, the zero offset and the missing load-turn `START` one at a time.
- Fidelity: D1 to D5 are carried by cards 1, 2, 3, 4 with 5, and 6. Every Constraint is honoured: `BudgetExempt` is set only on the two Burler hand-backs; the provider seam, completion signal, daemon wakeups and test-speed constraints are honoured; docs land in each owning card; the test budget is unchanged (`internal/battenshed` holds 86 top-level tests against a row of 90 before card 3's one addition). Card 4 stays inside D4's prompt-offset sentence and AC12, and card 2 inside D2.
- Verify coverage: `./...` under the untagged and `integration` tiers runs every targeted package, including `internal/shedadapters`, `internal/battenshed` and `internal/shuttleengine`. The section follows the template's order.
