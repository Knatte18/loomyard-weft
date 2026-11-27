```yaml
lens: interactions
outcome: reported
```

Task: the interactions lens. I checked the pairs of decisions that act on shared state and judged the criteria and constraints against them.

**Pairs checked**
- D6 hold × D10 chase (a pause ends the chase): held.
- D6 hold × D4 pause-lift resume (the hold is excluded and stands through a `lym tender resume` with no slug): held.
- D7 orch relay × D10 severe post under `unattended`, for orch-classified severe rows other than "no settling action": **broken**.
- D7 orch relay of an agent question × D10's `question-&lt;id&gt;` post: **broken** (the same defect as the previous line).
- D3 halt count × the tackler path D4 keeps (halt entry, tackler restep settle record, tackler attempt): **broken**.
- D3 × D4 driver rows (a settle record naming its halt entry counts once): held.
- D3 × D6 (an arbiter continue counts nothing; a handed-on Awaiting entry counts once; three handed-on escalations under `true` make the halt severe): held.
- D3 × D7 `halt:` key (a fold raises `repeats`, which gives a new key, a new count and a new post): held.
- D4 pause-lift resume × D5 park wait (the verb D4 uses, and whether it passes no-wait): **broken**, because D4 does not say.
- D5 × batten's child start and resume (no-wait passed, `ErrChildNotParked` retried): held.
- D7 pane relay × D12 slack button off × D10 severe post: **broken**.
- D8 reminders × D10 stalls with no halt (question, dead driver): **broken**.
- D8 keyless dedup × D8 "a new halt notifies again": **broken**, LOW.
- D2 "every hand-back carries `settle: orch`" × D7 hand-back with `settle: operator`: **broken**, LOW wording.
- D1 refused value read as `false` × D7, D12 and D6 at `false`: held.
- D2 at `false` × D4 rows at `false` × D7 stamps: held.
- D6 Bouncer with forced `blocking_escalates` × D6 arbiter rules × D9 narrower reading: held (D9 defers to D6).
- D6 hard-cap handed on × D2 hard-cap row: held.
- D10 question chase × D11 quiet-notice skip: held.
- D10 "names `lym loom resume`" × D5 wait: held.
- D6 held line × D7 severe post for a third handed-on escalation (a quiet line and a severe post): held; two lines, one of them quiet.
- The constraints (config-layers, config-key-names, agent-addressing, daemon-wakeups, leaf-packages) against D1, D7, D10 and D2: held.
- The criteria D2, D3, D7, D8 and D10 against the decisions they touch: held, except for the gaps below.

**Findings**

1. MEDIUM, consistency, `_lym/discussion/decision-record.md:253-256` (D7 × D10)
   - Summary: under `unattended` the orch relays its own severe classes, but D10 still chases the halt and then posts it as severe a second time.
     - The classes are "a run that cannot finish within its record" and "a step only the operator can take".
     - The run stays halted under `settle: orch` with the same `halt:` key.
     - D10 ends a chase only on a transition, a pause, an answer or a revive, never on the orch's relay (lines 317-318).
     - So one chase interval later D10 posts the same halt as severe again.
     - An agent question the orch relays as severe gets a second severe post too, keyed `question-&lt;id&gt;`.
     - The fix of round 1's F21 covered only "no settling action open".
     - This breaks D7's "one halt gets one severe post" (line 256).
     - D8's mention dedup holds only within `slack.remind_after_min`, while D10 posts two chase intervals after the halt.
   - Evidence: decision-record.md:253-258 and 311-318.

2. MEDIUM, design, `_lym/discussion/decision-record.md:131` (D3 × the tackler path)
   - Summary: D3 counts a tackler resume from its tackler attempt "since the tackler's attempts are kept apart from the driver's settle records". That premise is false.
     - `tackle` writes a settle record with action `SettleRestep` and outcome "tackler resume", plus a `SettleSpawn` record.
     - A tackler-resumed blocked halt therefore counts up to three times:
       - its halt entry;
       - a restep settle record, which D3 counts as "a re-step after a fault";
       - its tackler attempt.
     - A row reaches "third halt at the row" at its first or second real stop.
   - Evidence: internal/sheddriver/drive.go:491 and :533 (the `d.settle(stop, shedrun.SettlerTackler, …)` calls); internal/shedrun/settle.go:43-52.

3. MEDIUM, design, `_lym/discussion/decision-record.md:167-172` (D4 × D5)
   - Summary: D4 has the dobby resume a pause hand-back once the pause lifts, but names neither the verb it runs nor its no-wait option.
     - D5 makes `lym loom resume` and `lym loom start` wait up to the keep-warm cap.
     - D5's no-blocking claim covers only batten's child start and resume (lines 191-194).
     - A lift resume through the verb would block a dobby tick on a driver that has not yet parked.
   - Evidence: decision-record.md:165-172 and 183-196; internal/dobbyengine/passes.go:142-163 (`liftPending` gives a pass only to a drivable run).

4. MEDIUM, consistency, `_lym/discussion/decision-record.md:277-278` (D8 × D10)
   - Summary: D10 says D8's reminders cover a stall it posts as severe (line 316), but D8's reminders run "until the run leaves the halt".
     - A question stall or a dead-driver stall has no halt: the child is running.
     - So the reminder's end is undefined: it either never stops or stops at once.
     - D10's own end events (an answer, a settle, a revive) are not named as reminder ends.
   - Evidence: decision-record.md:277-278, 311-318; internal/dobbyengine/waitreminder.go:95 (`readWait` keys on a reminded row).

5. LOW, consistency, `_lym/discussion/decision-record.md:358` (D7 × D12 × D10)
   - Summary: while the dobby's orch has its slack button off, the orch relays in its pane only halts whose notice reads `settle: operator`.
     - D10's escalated severe post of a stall is no such notice.
     - The relay drops it (outbound.go:199-200), so under `unattended` it reaches neither Slack nor the pane.
     - D12's warning names only the halts lym posts.
   - Evidence: decision-record.md:259-260, 316, 355-358; internal/slackengine/outbound.go:199-201.

6. LOW, consistency, `_lym/discussion/decision-record.md:273` (D8)
   - Summary: "a post with no key dedups on the slug alone" collides with "a new halt of the same run … notifies again".
     - Suppose a keyless orch relay under `true` (a scope change or a cancel) arrives within `slack.remind_after_min` of a keyed dobby post for a different halt of the slug.
     - If the dedup counts the keyed post, a distinct halt notifies no phone.
     - The record does not say whether keyless posts dedup only against other keyless posts.
   - Evidence: decision-record.md:271-274, 257-258.

7. LOW, consistency, `_lym/discussion/decision-record.md:115` (D2 × D7)
   - Summary: "Every other stop is handed back with its verdict, `orch` or `operator` (D7), so every hand-back to the orch carries `settle: orch`" contradicts itself.
     - D7's hand-back notice also carries `settle: operator` (line 243).
     - It should say that a hand-back the table does not make severe carries `settle: orch`.
   - Evidence: decision-record.md:114-115, 238-243.

Tool calls: none.
