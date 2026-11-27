```yaml
lens: interactions
outcome: reported
```

Interactions lens: I ran each pair of decisions that act on the same state against each other, then checked the acceptance criteria and constraints against the decisions they touch.

**Pairs checked:**
- D1 × D6, autonomy `false` against the arbiter's new accept: broken. Under `false` the arbiter accepts deferred open BLOCKING findings.
- D1 × D2, the level against the table's columns: broken. The table has no `false` column.
- D1 × D4, `false` meaning "as today" against the new lym settles: held for the mechanical retries. It is the same gap as the D1 × D2 finding.
- D1 × D7/D10/D12, a refused value reading `false` against the Slack paths: held. D1's once-per-episode notice covers it.
- D2 × D3, verdict order against the halt count, board-claim bound one: held, giving lym, then orch, then operator.
- D2 × D3, rows whose operator column is "—" against today's class-5 repeating stop: broken.
- D2 × D6 × D3, how the arbiter's `budget` bound is counted: broken. D6 bounds it per segment, D3 over the row's whole history.
- D2 hard-cap row × D6 bound, nothing lands an open BLOCKING finding unattended: broken.
- D2 dead-driver row × D2's "bounds come from the table": broken (LOW).
- D2 scope-change and "no settling action" rows × D7's list of what the orch relays itself: broken.
- D3, "a re-step after a fault counts", × D3/D2, a dead driver counts nothing, with the existing interrupted-reinvoke re-step: broken.
- D3 × D4, bounce budget, gate attempts and pause: held.
- D3 × D6, arbiter continue and accept count nothing, a hand-on counts: held per halt. The bound part is broken, filed under the `budget` bound pair.
- D4 pause auto-resume × D6/D9, the orch's hold by pause: broken.
- D4 × D5, the dobby's resume through batten's non-waiting path: held. The caller retries on `ErrChildNotParked`, `internal/battencli/wire.go:510-516`.
- D4 × D10, pause hand-back against the chase: held, since a pause excludes the chase.
- D5 × push-retry-not-halt's wait record × D3, where a network wait counts nothing: held.
- D6 × D9, unattended hold and narrower reading: held.
- D6 × D10, the orch's hold pause ends the chase: held. The lift side is filed under the pause auto-resume pair.
- D7 × D8, restart double post against mention dedup: held.
- D7 × D10, the chase's severe post against the orch's own relay of "no settling action open" under `unattended`: broken.
- D7 × D12, posts dropped while the button is off: held, as D12 bounds it.
- D7 × the role's class-5 `lym selfreport create` filing: broken.
- D8 × D11, Raddle-Write reminders with no mention: held.
- D13 × D7, who classifies a severe halt: held.
- Acceptance criteria D2 ("at each level"), D6 ("accepted under `false` and `true`") and D3 ("arbiter accept counts nothing"): the D2 and D6 criteria inherit the breaks above. D3 holds.
- Constraints PATTERN-shed-driver, daemon-wakeups, agent-addressing and tolerant-records against D4, D5, D7, D8 and D10: held.

**Findings:**

1. **BLOCKING, design.** `decision-record.md` D6 (lines 181-182) against D2's hard-cap row (line 83). D6's bound says "nothing lands a round with an open BLOCKING finding under `unattended`". D2 keeps the arbiter "as today" for cause `hard-cap`. At hard-cap today the arbiter accepts, carrying open findings: the study gives accept as the only open decision (`stop-classes-2026-11-27.md:128`) and records "lym 17 (arbiter accept)" (`:52`). So under `unattended` the arbiter lands open BLOCKING findings at a hard cap, and D6's bound is false.

2. **BLOCKING, design.** D6 (lines 170-172) against D1 (line 43). D1 says `false` is "no autonomy, as today". D6 has the arbiter accept a green round with deferred open BLOCKING findings "under `false` and `true`", with no orch or operator. Today `Eligibility` refuses `budget`, and the stop goes to the parent (`internal/arbiterstop/arbiterstop.go:79-121`). At level `false` the orch relays that stop to the operator. D6 removes that operator gate at `false` and states no bound for doing so. Acceptance criterion D6 (line 356) pins the accept under `false`.

3. **MEDIUM, consistency.** D2 (lines 67, 77-95) and acceptance criterion D2 (line 341) against D1 (lines 41-45). The table returns a verdict "for a class, an autonomy level and the halt count", and the criterion pins every class "at each level". But the table has only `true` and `unattended` columns, so no verdict is defined for `false`. That leaves open whether D4's lym settles and D7's `settle:` stamp apply at `false`.

4. **MEDIUM, design.** D2's table (lines 82, 89-91) against role-autonomy class 5 (`contracts/orchrole/role-autonomy.md:38,43`). The rows for stale build, `driver_not_parked`, dead driver or API error, and permission dialog have "—" in the operator column. Under D2's verdict order they never reach the operator at any halt count. Today any third halt at a row is severe. The Auto-mode assumption (line 321) says the severe classes stay as today under `true`. D7 (line 200) limits what the orch relays itself to a scope change and "no settling action open", so a repeating stop of these classes reaches no one.

5. **MEDIUM, consistency.** D2 (line 92) against D7 (lines 200-201). The table gives a scope change or a cancel the operator verdict under `true`, which would make lym stamp `settle: operator` and post it. D7 says the orch itself relays "a scope change under `true`". D2's "no settling action open" row (line 94) has the same conflict with D7's relay list. No driver stop carries such a class, so either both post or neither does. D7's list also omits the cancel, which the line-321 assumption keeps severe under `true`.

6. **MEDIUM, consistency.** D6 ("once per review segment", line 174) against D3 ("lym's bound for a class counts lym's settles of that class at the row over the same history", lines 119-120) and D2's `budget` row (line 84). A second review segment at the same row, for example after a goto, gets a continue under D6 but none under D3's count over the whole history. The record does not say which count bounds the arbiter.

7. **MEDIUM, design.** D3 (lines 110 and 116) against the existing interrupted-reinvoke row. "A re-step after a fault" counts toward the halt count, while "a dead driver ... is no stop at a row" counts nothing. But a crash wake re-steps an interrupted row through settle records of action `restep`, and `wakeRestartGap` re-arms that count after each wake (`internal/sheddriver/driverules.go:295`, `internal/sheddriver/drive.go:809`). Those records count under D3's rule, so wakes after a crash can raise the row's count and make an unrelated later stop severe early.

8. **MEDIUM, design.** D4 (lines 139-141) against D9 (lines 233-234) and D6 (line 179). The unattended orch "holds" a run by pausing it, for a cancel it would make or an overtaken entry. A run paused while its driver runs gets a pause hand-back as its last stop. When that pause lifts, the dobby resumes the run. A blanket `lym tender resume` with no slug lifts every live entry (`internal/tenderengine/resume.go:23`), so it auto-resumes runs the orch held for the operator's call. D10 (line 253) has already ended the chase on that pause, so nothing re-raises it.

9. **LOW, consistency.** D7 (line 200) against D10 (line 251) under `unattended`. The orch relays "a halt with no settling action open" itself, which leaves the run halted with `settle: orch`. One chase interval later, D10's dobby posts the same halt as severe, outside D8's per-slug mention window when `beam.halt_chase_min` × 2 exceeds `slack.remind_after_min`. The operator gets two severe posts and two mentions for one halt.

10. **LOW, design.** D7 (lines 200-202) against role-autonomy (line 44, "File the bug a repeating stop shows with `lym selfreport create` before you relay it"). Repeating stops now carry `settle: operator`, which lym posts and the orch never relays. Nothing in D7 or D9 says who files the self-report for them.

11. **LOW, consistency.** D2's dead-driver row (line 90, "within today's caps") against D2 (line 69: "the driver's and the dobby's rules for a class lym settles take their bound from the table"). The row's bound is not in the table.

Tool calls: none; the inherited context held everything this lens needed.
