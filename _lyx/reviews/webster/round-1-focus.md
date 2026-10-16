---
round: 1
focus:
  - "internal/reedcli/watchdog.go: does the daemon's lock release, post-release session re-check and respawn ordering leave no window where a live hub ends up with no watchdog, and does a stop-woken idle cycle exit without respawning?"
  - "internal/reedengine/lifecycle.go and watchdog.go: is the stop marker written atomically (temp file renamed onto the signal path), and does a cold boot's truncating touch turn a pending stop back into a discover as the plan's Shared Decisions state?"
  - "internal/reedcli/up.go and cli.go: is the bounded poll of the watchdog lock in the down verb correctly bounded, does it handle a lock that never frees or a missing daemon, and does it match card 5's Intent?"
  - "internal/shuttleengine/claudeengine/command.go: does the env-file re-prepend of LYX_BIN's directory behave correctly per OS (quoting, path separators, an empty or unset LYX_BIN) as card 2 specifies?"
  - "Test economy across the new reedcli and reedengine tests: do the hermetic stop, re-check and replaced-ordering rows overlap existing watchdog_test.go or integration tests, and does any new behavior lack coverage?"
  - "Doc comments and package docs (doc.go in agentname, reedengine, reedcli, claudeengine) and the three sandbox suite pre-condition bullets: do they follow the repository's comment conventions and match the changed behavior?"
---

Seed focus from the plan's six cards. The range is the branch's own commits over the merge base with `product.parent`; the highest-risk code is the watchdog stop and re-check path (cards 3 to 5).
