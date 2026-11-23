loom halted failed at Code-Burler-Review

```
shedadapters: Code-Burler-Review (burler-review): round 1: two consecutive died/timeout outcomes (attempt 1 outcome died review never started (seatengine: seat "code-burler-review": start: shuttle: start: shuttle: the provider never became ready — run dir /home/knatte/Code/loomyard-LYMHUB/rc-button-ignores-org-policy/.lym/shuttle/6987d1e8d8ed23e06bab67b4ce4dfc1d, strand "a5b5d26142b561f1c22210e5cd7dcfe6", outcome "died"; the last pane capture was saved to startup-capture.txt; the strand was removed: shuttle: deliver the prompt after loading skills: shuttle: the session is not idle: strand "a5b5d26142b561f1c22210e5cd7dcfe6": no input box is on screen; a menu or picker may be open; retry once the session is idle. The pane's last lines:
  ⎿  Loaded skill
● Skill(scribe:golang-testing)
  ⎿  Loaded skill
● ok scribe:prose, scribe:code-quality,
  scribe:testing, scribe:golang-comments,
  scribe:golang-testing
✻ Sautéed for 3s · done 12:15
 ly:rc-button-ignores-org-policy:code-burler-review ─
❯ 
─────────────────────────────────────────────────────
  ⏵⏵ bypass permissions on (shift+tab to cycle) · …); attempt 1b outcome died review never started (seatengine: seat "code-burler-review": start: shuttle: start: shuttle: the provider never became ready — run dir /home/knatte/Code/loomyard-LYMHUB/rc-button-ignores-org-policy/.lym/shuttle/71317120df8bec826140dc2c71a72637, strand "4b87d5f50dff2e1accb27ec43b20cfe5", outcome "died"; the last pane capture was saved to startup-capture.txt; the strand was removed: shuttle: deliver the prompt after loading skills: shuttle: the session is not idle: strand "4b87d5f50dff2e1accb27ec43b20cfe5": no input box is on screen; a menu or picker may be open; retry once the session is idle. The pane's last lines:
  ⎿  Loaded skill
● Skill(scribe:golang-testing)
  ⎿  Loaded skill
● ok scribe:prose, scribe:code-quality,
  scribe:testing, scribe:golang-comments,
  scribe:golang-testing
✻ Churned for 5s · done 12:24
 ly:rc-button-ignores-org-policy:code-burler-review ─
❯ 
─────────────────────────────────────────────────────
  ⏵⏵ bypass permissions on (shift+tab to cycle) · …)): shuttle: the provider never became ready
```

producer: Code-Burler-Review
state: failed
anomaly: producer-hard-failure
history_entries: 8
trace_file: /home/knatte/Code/loomyard-LYMHUB/rc-button-ignores-org-policy/.lym/logs/trace-20261123T111134Z-40c87630978ef298-2555251.log

history:
- Preflight / done / 2026-11-23T10:22:57Z
- Record-Write / done / 2026-11-23T10:23:18Z
- Drift-Baseline / skipped / 2026-11-23T10:23:28Z
- Drift-Await / skipped / 2026-11-23T10:23:52Z
- Drift-Expect / skipped / 2026-11-23T10:24:03Z
- Darn-Write / done / 2026-11-23T11:02:02Z
- Drift-Verify / skipped / 2026-11-23T11:02:29Z
- Code-Bouncer / stuck / 2026-11-23T11:11:03Z / routed to Code-Burler-Review
