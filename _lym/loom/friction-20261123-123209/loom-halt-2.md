loom halted failed at Code-Burler-Review

```
shedadapters: Code-Burler-Review (burler-review): round 1: two consecutive died/timeout outcomes (attempt 1 outcome died review never started (seatengine: seat "code-burler-review": start: shuttle: start: shuttle: the provider never became ready — run dir /home/knatte/Code/loomyard-LYMHUB/verify-budget-notice-no-slug/.lym/shuttle/c7790ea1659a40883db5ba68f91c2898, strand "6fc05936b291bf598fcf3701b359acc1", outcome "died"; the last pane capture was saved to startup-capture.txt; the strand was removed: shuttle: deliver the prompt after loading skills: shuttle: the session is not idle: strand "6fc05936b291bf598fcf3701b359acc1": no input box is on screen; a menu or picker may be open; retry once the session is idle. The pane's last lines:
  ⎿  Loaded skill
● Skill(scribe:golang-testing)
  ⎿  Loaded skill
● ok scribe:prose, scribe:code-quality,
  scribe:testing, scribe:golang-comments,
  scribe:golang-testing
✻ Baked for 3s · done 12:30
 ly:verify-budget-notice-no-slug:code-burler-review ─
❯ 
─────────────────────────────────────────────────────
  ⏵⏵ bypass permissions on (shift+tab to cycle) · …); attempt 1b outcome died review never started (seatengine: seat "code-burler-review": start: shuttle: start: shuttle: the provider never became ready — run dir /home/knatte/Code/loomyard-LYMHUB/verify-budget-notice-no-slug/.lym/shuttle/950175ba133702bbea14e7330f5e28bf, strand "8046aba5567f7d0e739687af270f51d4", outcome "died"; the last pane capture was saved to startup-capture.txt; the strand was removed: shuttle: deliver the prompt after loading skills: shuttle: the session is not idle: strand "8046aba5567f7d0e739687af270f51d4": no input box is on screen; a menu or picker may be open; retry once the session is idle. The pane's last lines:
  ⎿  Loaded skill
● Skill(scribe:golang-testing)
  ⎿  Loaded skill
● ok scribe:prose, scribe:code-quality,
  scribe:testing, scribe:golang-comments,
  scribe:golang-testing
✻ Crunched for 3s · done 12:38
 ly:verify-budget-notice-no-slug:code-burler-review ─
❯ 
─────────────────────────────────────────────────────
  ⏵⏵ bypass permissions on (shift+tab to cycle) · …)): shuttle: the provider never became ready
```

producer: Code-Burler-Review
state: failed
anomaly: producer-hard-failure
history_entries: 8
trace_file: /home/knatte/Code/loomyard-LYMHUB/verify-budget-notice-no-slug/.lym/logs/trace-20261123T112534Z-6a312f3086395dfd-2793244.log

history:
- Preflight / done / 2026-11-23T10:51:20Z
- Record-Write / done / 2026-11-23T10:51:47Z
- Drift-Baseline / skipped / 2026-11-23T10:52:00Z
- Drift-Await / skipped / 2026-11-23T10:52:17Z
- Drift-Expect / skipped / 2026-11-23T10:52:28Z
- Darn-Write / done / 2026-11-23T10:59:07Z
- Drift-Verify / skipped / 2026-11-23T10:59:29Z
- Code-Bouncer / stuck / 2026-11-23T11:08:23Z / routed to Code-Burler-Review
