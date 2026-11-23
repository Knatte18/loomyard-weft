loom halted failed at Code-Burler-Fix

```
shedadapters: Code-Burler-Fix (burler-fix): round 1: two consecutive died/timeout outcomes (attempt 1 outcome died fix never started (seatengine: seat "code-burler-fix": start: shuttle: start: shuttle: the provider never became ready — run dir /home/knatte/Code/loomyard-LYMHUB/prompt-not-typed-after-skills/.lym/shuttle/7d150198f50e19bc3f07ef00f0cd750b, strand "21060b64557675e00578910898839194", outcome "died"; the last pane capture was saved to startup-capture.txt; the strand was removed: shuttle: deliver the prompt after loading skills: shuttle: the session is not idle: strand "21060b64557675e00578910898839194": no input box is on screen; a menu or picker may be open; retry once the session is idle. The pane's last lines:
  ⎿  Loaded skill
● Skill(scribe:golang-testing)
  ⎿  Loaded skill
● ok scribe:prose, scribe:code-quality,
  scribe:testing, scribe:golang-comments,
  scribe:golang-testing
✻ Brewed for 2s · done 12:30
── ly:prompt-not-typed-after-skills:code-burler-fix ─
❯ 
─────────────────────────────────────────────────────
  ⏵⏵ bypass permissions on (shift+tab to cycle) · …); attempt 1b outcome died fix never started (seatengine: seat "code-burler-fix": start: shuttle: start: shuttle: the provider never became ready — run dir /home/knatte/Code/loomyard-LYMHUB/prompt-not-typed-after-skills/.lym/shuttle/1ecee2d5d96293a89bd4e4abcbb05151, strand "7f11fd11ed51242c198704d6f643ce54", outcome "died"; the last pane capture was saved to startup-capture.txt; the strand was removed: shuttle: deliver the prompt after loading skills: shuttle: the session is not idle: strand "7f11fd11ed51242c198704d6f643ce54": no input box is on screen; a menu or picker may be open; retry once the session is idle. The pane's last lines:
  ⎿  Loaded skill
● Skill(scribe:golang-testing)
  ⎿  Loaded skill
● ok scribe:prose, scribe:code-quality,
  scribe:testing, scribe:golang-comments,
  scribe:golang-testing
✻ Brewed for 2s · done 12:39
── ly:prompt-not-typed-after-skills:code-burler-fix ─
❯ 
─────────────────────────────────────────────────────
  ⏵⏵ bypass permissions on (shift+tab to cycle) · …)): shuttle: the provider never became ready
```

producer: Code-Burler-Fix
state: failed
anomaly: producer-hard-failure
history_entries: 9
trace_file: /home/knatte/Code/loomyard-LYMHUB/prompt-not-typed-after-skills/.lym/logs/trace-20261123T112639Z-5dfa0dae153d6446-2801170.log

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
