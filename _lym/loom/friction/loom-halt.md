loom halted failed at Code-Burler-Review

```
shedadapters: Code-Burler-Review (burler-review): round 1: two consecutive died/timeout outcomes (attempt 1 outcome died review never started (seatengine: seat "code-burler-review": start: shuttle: start: shuttle: the provider never became ready — run dir /home/knatte/Code/loomyard-LYMHUB/intake-checks-already-fixed/.lym/shuttle/d5d6b8b42bcf05843fd13418e9f8e913, strand "7daea1c74e937ee94c7b87b8c0357864", outcome "died"; the last pane capture was saved to startup-capture.txt; the strand was removed: shuttle: deliver the prompt after loading skills: shuttle: the session is not idle: strand "7daea1c74e937ee94c7b87b8c0357864": no input box is on screen; a menu or picker may be open; retry once the session is idle. The pane's last lines:
  ⎿  Loaded skill
● Skill(scribe:golang-testing)
  ⎿  Loaded skill
● ok scribe:prose, scribe:code-quality,
  scribe:testing, scribe:golang-comments,
  scribe:golang-testing
✻ Baked for 4s · done 12:01
─ ly:intake-checks-already-fixed:code-burler-review ─
❯ 
─────────────────────────────────────────────────────
  ⏵⏵ bypass permissions on (shift+tab to cycle) · …); attempt 1b outcome died review never started (seatengine: seat "code-burler-review": start: shuttle: start: shuttle: the provider never became ready — run dir /home/knatte/Code/loomyard-LYMHUB/intake-checks-already-fixed/.lym/shuttle/ca49f8861bd934e3452527bb3d354bfe, strand "960a6243a2be4940f9c8db904a60d03d", outcome "died"; the last pane capture was saved to startup-capture.txt; the strand was removed: shuttle: deliver the prompt after loading skills: shuttle: the session is not idle: strand "960a6243a2be4940f9c8db904a60d03d": no input box is on screen; a menu or picker may be open; retry once the session is idle. The pane's last lines:
  ⎿  Loaded skill
● Skill(scribe:golang-testing)
  ⎿  Loaded skill
● ok scribe:prose, scribe:code-quality,
  scribe:testing, scribe:golang-comments,
  scribe:golang-testing
✻ Sautéed for 5s · done 12:10
─ ly:intake-checks-already-fixed:code-burler-review ─
❯ 
─────────────────────────────────────────────────────
  ⏵⏵ bypass permissions on (shift+tab to cycle) · …)): shuttle: the provider never became ready
```

producer: Code-Burler-Review
state: failed
anomaly: producer-hard-failure
history_entries: 8
trace_file: /home/knatte/Code/loomyard-LYMHUB/intake-checks-already-fixed/.lym/logs/trace-20261123T105728Z-6c54a536b6b12cd6-1696681.log

history:
- Preflight / done / 2026-11-23T10:37:46Z
- Record-Write / done / 2026-11-23T10:37:59Z
- Drift-Baseline / skipped / 2026-11-23T10:38:24Z
- Drift-Await / skipped / 2026-11-23T10:38:32Z
- Drift-Expect / skipped / 2026-11-23T10:39:00Z
- Darn-Write / done / 2026-11-23T10:48:20Z
- Drift-Verify / skipped / 2026-11-23T10:48:42Z
- Code-Bouncer / stuck / 2026-11-23T10:57:14Z / routed to Code-Burler-Review
