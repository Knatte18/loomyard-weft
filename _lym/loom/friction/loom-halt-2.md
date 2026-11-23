loom halted failed at Code-Burler-Fix

```
shedadapters: Code-Burler-Fix (burler-fix): round 1: two consecutive died/timeout outcomes (attempt 1 outcome died fix never started (seatengine: seat "code-burler-fix": start: shuttle: start: shuttle: the provider never became ready — run dir /home/knatte/Code/loomyard-LYMHUB/prompt-not-typed-after-skills/.lym/shuttle/a1447df1faf722641e70a19a553df4e2, strand "7ca7cbfe4591c42877fda968c0a1258e", outcome "died"; the last pane capture was saved to startup-capture.txt; the strand was removed: shuttle: deliver the prompt after loading skills: shuttle: the session is not idle: strand "7ca7cbfe4591c42877fda968c0a1258e": no input box is on screen; a menu or picker may be open; retry once the session is idle. The pane's last lines:
  ⎿  Loaded skill
● Skill(scribe:golang-testing)
  ⎿  Loaded skill
● ok scribe:prose, scribe:code-quality,
  scribe:testing, scribe:golang-comments,
  scribe:golang-testing
✻ Cooked for 2s · done 12:48
── ly:prompt-not-typed-after-skills:code-burler-fix ─
❯ 
─────────────────────────────────────────────────────
  ⏵⏵ bypass permissions on (shift+tab to cycle) · …); attempt 1b outcome died fix never started (seatengine: seat "code-burler-fix": start: shuttle: start: shuttle: the provider never became ready — run dir /home/knatte/Code/loomyard-LYMHUB/prompt-not-typed-after-skills/.lym/shuttle/4e5f8b8b4d261fb2fa785ebfb72f7dae, strand "da6cb71e1e9620bb02152ca71d1b9114", outcome "died"; the last pane capture was saved to startup-capture.txt; the strand was removed: shuttle: deliver the prompt after loading skills: shuttle: the session is not idle: strand "da6cb71e1e9620bb02152ca71d1b9114": no input box is on screen; a menu or picker may be open; retry once the session is idle. The pane's last lines:
  ⎿  Loaded skill
● Skill(scribe:golang-testing)
  ⎿  Loaded skill
● ok scribe:prose, scribe:code-quality,
  scribe:testing, scribe:golang-comments,
  scribe:golang-testing
✻ Cooked for 3s · done 12:56
── ly:prompt-not-typed-after-skills:code-burler-fix ─
❯ 
─────────────────────────────────────────────────────
  ⏵⏵ bypass permissions on (shift+tab to cycle) · …)): shuttle: the provider never became ready
```

producer: Code-Burler-Fix
state: failed
anomaly: producer-hard-failure
history_entries: 9
trace_file: /home/knatte/Code/loomyard-LYMHUB/prompt-not-typed-after-skills/.lym/logs/trace-20261123T114334Z-15408e753f25319d-3167172.log

history:
- Preflight / done / 2026-11-23T10:58:19Z
- Record-Write / done / 2026-11-23T10:58:39Z
- Drift-Baseline / skipped / 2026-11-23T10:59:08Z
- Drift-Await / skipped / 2026-11-23T10:59:28Z
- Drift-Expect / skipped / 2026-11-23T10:59:53Z
- Darn-Write / done / 2026-11-23T11:14:26Z
- Drift-Verify / skipped / 2026-11-23T11:14:44Z
- Code-Bouncer / stuck / 2026-11-23T11:23:18Z / routed to Code-Burler-Review
- Code-Burler-Review / stuck / 2026-11-23T11:26:25Z / routed to Code-Burler-Fix
