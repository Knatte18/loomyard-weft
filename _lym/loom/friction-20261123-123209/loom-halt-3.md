loom halted failed at Code-Burler-Review

```
shedadapters: Code-Burler-Review (burler-review): round 1: two consecutive died/timeout outcomes (attempt 1 outcome died review never started (seatengine: seat "code-burler-review": start: shuttle: start: shuttle: the provider never became ready — run dir /home/knatte/Code/loomyard-LYMHUB/verify-budget-notice-no-slug/.lym/shuttle/ef00ba0d9b8f14f5834173eb1b8fa783, strand "9d138622f1faac7297c43a981878d065", outcome "died"; the last pane capture was saved to startup-capture.txt; the strand was removed: shuttle: deliver the prompt after loading skills: shuttle: the session is not idle: strand "9d138622f1faac7297c43a981878d065": no input box is on screen; a menu or picker may be open; retry once the session is idle. The pane's last lines:
  ⎿  Loaded skill
● Skill(scribe:golang-testing)
  ⎿  Loaded skill
● ok scribe:prose, scribe:code-quality,
  scribe:testing, scribe:golang-comments,
  scribe:golang-testing
✻ Churned for 5s · done 12:51
 ly:verify-budget-notice-no-slug:code-burler-review ─
❯ 
─────────────────────────────────────────────────────
  ⏵⏵ bypass permissions on (shift+tab to cycle) · …); attempt 1b outcome died review never started (seatengine: seat "code-burler-review": start: shuttle: start: shuttle: the provider never became ready — run dir /home/knatte/Code/loomyard-LYMHUB/verify-budget-notice-no-slug/.lym/shuttle/95e2dde7e87b703e4493675ce12bdb23, strand "7294a722f1d673499a68fb163df298a1", outcome "died"; the last pane capture was saved to startup-capture.txt; the strand was removed: shuttle: deliver the prompt after loading skills: shuttle: the session is not idle: strand "7294a722f1d673499a68fb163df298a1": no input box is on screen; a menu or picker may be open; retry once the session is idle. The pane's last lines:
  ⎿  Loaded skill
● Skill(scribe:golang-testing)
  ⎿  Loaded skill
● ok scribe:prose, scribe:code-quality,
  scribe:testing, scribe:golang-comments,
  scribe:golang-testing
✻ Brewed for 5s · done 12:59
 ly:verify-budget-notice-no-slug:code-burler-review ─
❯ 
─────────────────────────────────────────────────────
  ⏵⏵ bypass permissions on (shift+tab to cycle) · …)): shuttle: the provider never became ready
```

producer: Code-Burler-Review
state: failed
anomaly: producer-hard-failure
history_entries: 8
trace_file: /home/knatte/Code/loomyard-LYMHUB/verify-budget-notice-no-slug/.lym/logs/trace-20261123T114708Z-57aff39486cefc4f-3197021.log

history:
- Preflight / done / 2026-11-23T10:51:20Z
- Record-Write / done / 2026-11-23T10:51:47Z
- Drift-Baseline / skipped / 2026-11-23T10:52:00Z
- Drift-Await / skipped / 2026-11-23T10:52:17Z
- Drift-Expect / skipped / 2026-11-23T10:52:28Z
- Darn-Write / done / 2026-11-23T10:59:07Z
- Drift-Verify / skipped / 2026-11-23T10:59:29Z
- Code-Bouncer / stuck / 2026-11-23T11:08:23Z / routed to Code-Burler-Review
