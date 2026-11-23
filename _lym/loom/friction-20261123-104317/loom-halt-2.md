loom halted failed at Code-Burler-Review

```
shedadapters: Code-Burler-Review (burler-review): round 1: two consecutive died/timeout outcomes (attempt 1 outcome died review never started (seatengine: seat "code-burler-review": start: shuttle: start: shuttle: the provider never became ready — run dir /home/knatte/Code/loomyard-LYMHUB/godocreflow-worsens-findings/.lym/shuttle/b641fa7a0dd7ddab81ec2185f59ef19c, strand "8a98b64a2f08c036172d94d23fc8b3e7", outcome "died"; the last pane capture was saved to startup-capture.txt; the strand was removed: shuttle: deliver the prompt after loading skills: shuttle: the session is not idle: strand "8a98b64a2f08c036172d94d23fc8b3e7": no input box is on screen; a menu or picker may be open; retry once the session is idle. The pane's last lines:
  ⎿  Loaded skill
● Skill(scribe:golang-testing)
  ⎿  Loaded skill
● ok scribe:prose, scribe:code-quality,
  scribe:testing, scribe:golang-comments,
  scribe:golang-testing
✻ Brewed for 5s · done 10:13
 ly:godocreflow-worsens-findings:code-burler-review ─
❯ 
─────────────────────────────────────────────────────
  ⏵⏵ bypass permissions on (shift+tab to cycle) · …); attempt 1b outcome died review never started (seatengine: seat "code-burler-review": start: shuttle: start: shuttle: the provider never became ready — run dir /home/knatte/Code/loomyard-LYMHUB/godocreflow-worsens-findings/.lym/shuttle/a63b3037389a2e9a107d325fd418d2d6, strand "06d2a76e1270ed20aba19e3c5b70de5c", outcome "died"; the last pane capture was saved to startup-capture.txt; the strand was removed: shuttle: deliver the prompt after loading skills: shuttle: the session is not idle: strand "06d2a76e1270ed20aba19e3c5b70de5c": no input box is on screen; a menu or picker may be open; retry once the session is idle. The pane's last lines:
  ⎿  Loaded skill
● Skill(scribe:golang-testing)
  ⎿  Loaded skill
● ok scribe:prose, scribe:code-quality,
  scribe:testing, scribe:golang-comments,
  scribe:golang-testing
✻ Worked for 5s · done 10:22
 ly:godocreflow-worsens-findings:code-burler-review ─
❯ 
─────────────────────────────────────────────────────
  ⏵⏵ bypass permissions on (shift+tab to cycle) · …)): shuttle: the provider never became ready
```

producer: Code-Burler-Review
state: failed
anomaly: producer-hard-failure
history_entries: 8
trace_file: /home/knatte/Code/loomyard-LYMHUB/godocreflow-worsens-findings/.lym/logs/trace-20261123T090915Z-86fdf01fba6e7977-2957331.log

history:
- Preflight / done / 2026-11-23T08:25:29Z
- Record-Write / done / 2026-11-23T08:25:45Z
- Drift-Baseline / skipped / 2026-11-23T08:26:07Z
- Drift-Await / skipped / 2026-11-23T08:26:23Z
- Drift-Expect / skipped / 2026-11-23T08:26:30Z
- Darn-Write / done / 2026-11-23T08:51:02Z
- Drift-Verify / skipped / 2026-11-23T08:51:13Z
- Code-Bouncer / stuck / 2026-11-23T08:59:59Z / routed to Code-Burler-Review
