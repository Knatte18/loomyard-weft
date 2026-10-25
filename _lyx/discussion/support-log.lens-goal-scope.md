```yaml
lens: goal-scope
outcome: reported
```

Task: review the umbrella-daemon decision record and support log against the board entry, through the goal-scope lens.

**Findings**

1. **MEDIUM. D7, the escalation chain.** Dobby's own failures skip the arbiter, which goes against an operator decision.
   - The record sends a refused start and a pass that keeps failing straight to the nearest orch.
   - The entry's operator decisions (2026-10-18, beam and orch apart) say: "What a beam's daemon cannot settle escalates upward: through the arbiter first, then to the beam's orch, or, when the beam has none, to the parent beam's orch."
   - The parent's 2026-10-25 message also says "batten and dobby included".
   - The record handles this through an auto-mode assumption and a rejected alternative, so it does not depart from the decision silently. Still, it is a departure from an operator decision, settled by the writer's own pick with no question sent to `ly:orch`.
   - Fix: send dobby's failures to the arbiter first, or ask `ly:orch` to confirm the departure.

2. **MEDIUM. D12 with D4, after a pause.** "Starts, lands, revives nothing in a paused hub", and "checks that each interrupted run came back", are not fully met.
   - Dobby keeps running passes under a pause. A pass that reaches Land while the pause covers it halts the row blocked, per the battenshed doc: "halts the row blocked with no notice … the way back is `lyx batten goto <slug> --to Land`".
   - The pass dobby runs once the entry is lifted cannot bring such a run back, because a blocked Land needs a goto by hand. The goal of landing with no orch turn is then missed exactly after a pause.
   - The entry's own recommendation was that dobby lands nothing in a paused hub. That points to holding passes at Land, or not entering Land at all, while the pause covers the run.

3. **LOW. D11.** The config landing check stays in Finalize, while the operator called it "a job for the Go run driver" (2026-10-13).
   The rationale is stated and the function is mechanical Go, which is the substance of what the operator asked. Acceptable, but the record should say outright that this departs from the entry's wording.

4. **LOW. D12, `tender` and dobby.** One case the entry saw is not addressed.
   - The entry records that "`tender resume` woke no Plan-Write strand that had a background fork … which dobby, following status files, could notice".
   - D12 relies on batten's existing respawn, revive and driver-dead checks. None of them catches a live strand that is idle with a background fork.
   - Either cover it, or state that it is out of scope.

5. **LOW. Scope, the orch rule left in place.** The hub-wide orch rule "Board tidy after a landing" stays until the parent removes it.
   - D9 drops the `landed` notice, which is that rule's trigger, so the rule becomes dead text and does not cause a double spawn.
   - Fine as an operator action. Consider having dobby's first tidy report, or the landing, remind the parent to remove it, so the rule does not go stale.

6. **LOW. D4, unlicensed CLI change.** Retiring `--window` and `--open-terminal`, and removing note `batten-window-default`, go beyond what the entry asks.
   They are defensible, since dobby starts every run, but they are additions the entry did not license and should be named as such.

7. **LOW. D5.** A refused first pass is not retried until the entry's `revised` time changes.
   A refusal caused by something passing, such as origin being unreachable for the board pull, then needs a board edit to clear it. The goal of starting ready tasks with no orch involved is weakened for causes that clear by themselves.

**Checked and held**

- Draft as a status, the README mark and readiness without notices are covered.
- `lyx beam start` and the call from `lyx orch start` are covered.
- The 2026-10-25 point that a run's parent always has a running dobby is met: every beam gets one from birth.
- The revision of every kind of batten notice is covered.
- The board-tidy agent every N landings, with N defaulting to 5, is covered.
- The arbiter fold-in, including the description of batten's rows, is covered.
- The "umbrella" to "beam" rename is covered.
- The fsnotify watch with a polling fallback is covered.
- Restart recovery through dobby alone is covered.
- No MVP phase or "add later" deferral is hidden in the text.
- Leaving batten's logic in batten is an explicit decision rather than a deferral, and matches "Batten drives one task's lifecycle as today".
- Building this is the right thing: it is the operator's settled direction.

I made no further tool calls.
