loom halted failed at Code-Burler-Review

```
shedadapters: Code-Burler-Review (burler-review): round 1: two consecutive died/timeout outcomes (attempt 1 outcome died review never started (seatengine: seat "code-burler-review": start: shuttle: start: shuttle: the provider never became ready — run dir /home/knatte/Code/loomyard-LYMHUB/rc-button-ignores-org-policy/.lym/shuttle/d5f12632648343792af39d574a97be0e, strand "37460e31c5125f74a3419bd3f8549860", outcome "died"; the last pane capture was saved to startup-capture.txt; the strand was removed: shuttle: deliver the prompt after loading skills: shuttle: the session is not idle: strand "37460e31c5125f74a3419bd3f8549860": no input box is on screen; a menu or picker may be open; retry once the session is idle. The pane's last lines:
  ⎿  Loaded skill
● Skill(scribe:golang-testing)
  ⎿  Loaded skill
● ok scribe:prose, scribe:code-quality,
  scribe:testing, scribe:golang-comments,
  scribe:golang-testing
✻ Crunched for 5s · done 12:33
 ly:rc-button-ignores-org-policy:code-burler-review ─
❯ 
─────────────────────────────────────────────────────
  ⏵⏵ bypass permissions on (shift+tab to cycle) · …); attempt 1b outcome died review never started (seatengine: seat "code-burler-review": start: shuttle: start: shuttle: the provider never became ready — run dir /home/knatte/Code/loomyard-LYMHUB/rc-button-ignores-org-policy/.lym/shuttle/60cfc02be1a1dd862206bc31b4a70302, strand "83504bfd3bf55731560e247cb1e3d9dc", outcome "died"; the last pane capture was saved to startup-capture.txt; the strand was removed: shuttle: deliver the prompt after loading skills: shuttle: the session is not idle: strand "83504bfd3bf55731560e247cb1e3d9dc": no input box is on screen; a menu or picker may be open; retry once the session is idle. The pane's last lines:
  ⎿  Loaded skill
● Skill(scribe:golang-testing)
  ⎿  Loaded skill
● ok scribe:prose, scribe:code-quality,
  scribe:testing, scribe:golang-comments,
  scribe:golang-testing
✻ Cooked for 5s · done 12:41
 ly:rc-button-ignores-org-policy:code-burler-review ─
❯ 
─────────────────────────────────────────────────────
  ⏵⏵ bypass permissions on (shift+tab to cycle) · …)): shuttle: the provider never became ready
```

producer: Code-Burler-Review
state: failed
anomaly: producer-hard-failure
history_entries: 8
trace_file: /home/knatte/Code/loomyard-LYMHUB/rc-button-ignores-org-policy/.lym/logs/trace-20261123T112840Z-c819b16805eeb151-2814599.log

history:
- Preflight / done / 2026-11-23T10:22:57Z
- Record-Write / done / 2026-11-23T10:23:18Z
- Drift-Baseline / skipped / 2026-11-23T10:23:28Z
- Drift-Await / skipped / 2026-11-23T10:23:52Z
- Drift-Expect / skipped / 2026-11-23T10:24:03Z
- Darn-Write / done / 2026-11-23T11:02:02Z
- Drift-Verify / skipped / 2026-11-23T11:02:29Z
- Code-Bouncer / stuck / 2026-11-23T11:11:03Z / routed to Code-Burler-Review
