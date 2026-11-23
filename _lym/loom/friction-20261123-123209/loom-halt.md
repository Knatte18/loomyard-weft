loom halted failed at Code-Burler-Review

```
shedadapters: Code-Burler-Review (burler-review): round 1: two consecutive died/timeout outcomes (attempt 1 outcome died review never started (seatengine: seat "code-burler-review": start: shuttle: start: shuttle: the provider never became ready — run dir /home/knatte/Code/loomyard-LYMHUB/verify-budget-notice-no-slug/.lym/shuttle/e8e4c384c457cc04b1ef4187e63bc010, strand "e372554846db1be65836d3c7315f51fe", outcome "died"; the last pane capture was saved to startup-capture.txt; the strand was removed: shuttle: deliver the prompt after loading skills: shuttle: the session is not idle: strand "e372554846db1be65836d3c7315f51fe": no input box is on screen; a menu or picker may be open; retry once the session is idle. The pane's last lines:
  ⎿  Loaded skill
● Skill(scribe:golang-testing)
  ⎿  Loaded skill
● ok scribe:prose, scribe:code-quality,
  scribe:testing, scribe:golang-comments,
  scribe:golang-testing
✻ Worked for 6s · done 12:13
 ly:verify-budget-notice-no-slug:code-burler-review ─
❯ 
─────────────────────────────────────────────────────
  ⏵⏵ bypass permissions on (shift+tab to cycle) · …); attempt 1b outcome died review never started (seatengine: seat "code-burler-review": start: shuttle: start: shuttle: the provider never became ready — run dir /home/knatte/Code/loomyard-LYMHUB/verify-budget-notice-no-slug/.lym/shuttle/101c6f1f9550736833fa80b390014fe7, strand "5347d705524267074933fa28e7ef8c3b", outcome "died"; the last pane capture was saved to startup-capture.txt; the strand was removed: shuttle: deliver the prompt after loading skills: shuttle: the session is not idle: strand "5347d705524267074933fa28e7ef8c3b": no input box is on screen; a menu or picker may be open; retry once the session is idle. The pane's last lines:
  ⎿  Loaded skill
● Skill(scribe:golang-testing)
  ⎿  Loaded skill
● ok scribe:prose, scribe:code-quality,
  scribe:testing, scribe:golang-comments,
  scribe:golang-testing
✻ Churned for 3s · done 12:21
 ly:verify-budget-notice-no-slug:code-burler-review ─
❯ 
─────────────────────────────────────────────────────
  ⏵⏵ bypass permissions on (shift+tab to cycle) · …)): shuttle: the provider never became ready
```

producer: Code-Burler-Review
state: failed
anomaly: producer-hard-failure
history_entries: 8
trace_file: /home/knatte/Code/loomyard-LYMHUB/verify-budget-notice-no-slug/.lym/logs/trace-20261123T110831Z-0633fb0219897048-2309320.log

history:
- Preflight / done / 2026-11-23T10:51:20Z
- Record-Write / done / 2026-11-23T10:51:47Z
- Drift-Baseline / skipped / 2026-11-23T10:52:00Z
- Drift-Await / skipped / 2026-11-23T10:52:17Z
- Drift-Expect / skipped / 2026-11-23T10:52:28Z
- Darn-Write / done / 2026-11-23T10:59:07Z
- Drift-Verify / skipped / 2026-11-23T10:59:29Z
- Code-Bouncer / stuck / 2026-11-23T11:08:23Z / routed to Code-Burler-Review
