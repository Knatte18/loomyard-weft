loom halted failed at Code-Burler-Review

```
shedadapters: Code-Burler-Review (burler-review): round 1: two consecutive died/timeout outcomes (attempt 1 outcome died review never started (seatengine: seat "code-burler-review": start: shuttle: start: shuttle: the provider never became ready — run dir /home/knatte/Code/loomyard-LYMHUB/intake-checks-already-fixed/.lym/shuttle/3776b4e7d4aa53f595541a73c2ea19d3, strand "090115d2bcf5e3d697ac0f4817689193", outcome "died"; the last pane capture was saved to startup-capture.txt; the strand was removed: shuttle: deliver the prompt after loading skills: shuttle: the session is not idle: strand "090115d2bcf5e3d697ac0f4817689193": no input box is on screen; a menu or picker may be open; retry once the session is idle. The pane's last lines:
  ⎿  Loaded skill
● Skill(scribe:golang-testing)
  ⎿  Loaded skill
● ok scribe:prose, scribe:code-quality,
  scribe:testing, scribe:golang-comments,
  scribe:golang-testing
✻ Worked for 4s · done 12:18
─ ly:intake-checks-already-fixed:code-burler-review ─
❯ 
─────────────────────────────────────────────────────
  ⏵⏵ bypass permissions on (shift+tab to cycle) · …); attempt 1b outcome died review never started (seatengine: seat "code-burler-review": start: shuttle: start: shuttle: the provider never became ready — run dir /home/knatte/Code/loomyard-LYMHUB/intake-checks-already-fixed/.lym/shuttle/a2f8f6861a50371790b54ec1239d7ae4, strand "2c0d6008b17383dd426e5aad8d018762", outcome "died"; the last pane capture was saved to startup-capture.txt; the strand was removed: shuttle: deliver the prompt after loading skills: shuttle: the session is not idle: strand "2c0d6008b17383dd426e5aad8d018762": no input box is on screen; a menu or picker may be open; retry once the session is idle. The pane's last lines:
  ⎿  Loaded skill
● Skill(scribe:golang-testing)
  ⎿  Loaded skill
● ok scribe:prose, scribe:code-quality,
  scribe:testing, scribe:golang-comments,
  scribe:golang-testing
✻ Churned for 5s · done 12:27
─ ly:intake-checks-already-fixed:code-burler-review ─
❯ 
─────────────────────────────────────────────────────
  ⏵⏵ bypass permissions on (shift+tab to cycle) · …)): shuttle: the provider never became ready
```

producer: Code-Burler-Review
state: failed
anomaly: producer-hard-failure
history_entries: 8
trace_file: /home/knatte/Code/loomyard-LYMHUB/intake-checks-already-fixed/.lym/logs/trace-20261123T111425Z-3ccd1cd8f7bda61c-2717397.log

history:
- Preflight / done / 2026-11-23T10:37:46Z
- Record-Write / done / 2026-11-23T10:37:59Z
- Drift-Baseline / skipped / 2026-11-23T10:38:24Z
- Drift-Await / skipped / 2026-11-23T10:38:32Z
- Drift-Expect / skipped / 2026-11-23T10:39:00Z
- Darn-Write / done / 2026-11-23T10:48:20Z
- Drift-Verify / skipped / 2026-11-23T10:48:42Z
- Code-Bouncer / stuck / 2026-11-23T10:57:14Z / routed to Code-Burler-Review
