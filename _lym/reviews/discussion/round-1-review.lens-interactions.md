```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. I checked every pair of decisions that act on the same state, and checked the acceptance criteria and constraints against the decisions they touch.

Pairs checked:
- D1 × D3, Stop count: held. The ring's claim resets the count to 0, so the count is known to the daemon. The woken turn's Stop can still claim the mail left past the budget, and that adds to the count under cap 6.
- D1 × D3, the woken turn's `stop_hook_active`: held. The rewake notification is enqueued with `stopHookActive:!0`, so the woken turn's first Stop may arrive with the flag true and creel does not reset the count. The ring has already reset it, so nothing is lost.
- D1 × D2, the shared seam renderer and its per-event keys (D1 drops `systemMessage`, D2 keeps it): held, no contradiction.
- D1 × D2, the tool-boundary claim resets the count while D1 continues the turn: held. This is the existing rule and D2 does not change it.
- D1 × Constraint "hook-read writes before it acknowledges": held. The duplicate after a failed ack is the stated exception, and D1's continuation does not widen it.
- D3 × Constraint "each message delivered once", ack-failure path: held. hook-wait exits 0, and Claude Code enqueues a wake only on exit 2 (`_tt`: `if(Ae.code===2)`), so the stderr never reaches the model.
- D3 × the existing bell's `ringAnswerWindow` and `wakeMarks`: held. The ring is answered on ack; on a failed ack the mark stays unanswered and the bell follows after 30 s.
- D3 × the budget-left remainder × `ringHook`'s `ids` equality: held. The leftover set differs from `mark.ids`, so a later ring or the woken turn's boundary takes it.
- D3 pause read × the worker's own pause check: held. `locate()` already excludes paused strands, and the D3 bound covers the remaining window.
- D3 × `PATTERN-tolerant-records`, older daemon or older waiter during a deploy: held, and both directions are stated (exit 0 then bell; unmarked waiter rung with no mail).
- D4 × D3, old-session settings meeting a new `hook-wait`: held. Settings and binary change together at relaunch, as D4's bound says.
- D3 × D5, `released` and `acknowledged` on the ring path: held.
- D3 × D5, a ring to an unmarked waiter: broken in coverage, see finding 2.
- D3/D4 × Goal "the operator once": not stated, see finding 1.
- Constraint "8 KiB fits every field" × D1/D2/D3: held. The first line handed out is capped at `hookFieldCap`, and the rewake stderr is not truncated.

Findings:

1. LOW, consistency, decision-record D3/D4 and Acceptance criteria. The Goal requires the operator to read each mail once, but neither D3 nor D4 states what the operator sees on a ring wake, and no criterion checks the operator's copy on the ring path.
   - Claude Code enqueues the wake as a `task-notification` whose summary is `rewakeSummary` (D4's `creel mail`), with the stderr as its trailing body.
   - The record never says whether the terminal shows that body or only the summary, so the ring path may show the operator zero copies, or two counting the summary.
   - D1 and D2 each state the operator's copy; D3 and D4 should do the same.
   - Evidence: binary `Bin({summary,body,...})` → `Di({summary:zt(e),trailing:\n${Xa(n)}})`, `mode:"task-notification"`; `_tt` sets `rt=We??G??"Stop hook feedback"`.

2. LOW, consistency, D5 against D3. D5 logs every claim, and the ring path only when it claims, but D3 keeps a ring to an unmarked waiter that claims nothing. That wake, a hook delivery attempt in the Goal's sense, gets no log line.
   - Evidence: decision-record D3 "rings an unmarked one ... as today, with no mail"; D5 "Every claim through the claim verb or the ring"; `ringHook` in `internal/creelengine/verbs.go:435` answers the ring without a claim.

Tool calls:
1. `grep` in `internal/creelengine/deliver.go` for `wakeHookStrand` and `locate`: to see whether the worker skips paused strands before ringing; the manifest had `wakeHookStrand` but not `locate`.
2. `grep` in the binary for `function Bin(`: to learn what the rewake notification carries, since the manifest showed only the call site in `_tt`.
3. Same grep with a wider window: the output was cut at a newline inside a template literal.
4. `dd` of `Bin` and `Di` in the binary: to read the notification's summary and body shape for finding 1.
