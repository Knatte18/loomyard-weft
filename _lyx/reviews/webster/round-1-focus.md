---
round: 1
focus:
  - "internal/orchengine PeekState: settle whether it truly takes no lock, creates no lock file and does no mkdir, and how it handles a missing, partial or mid-write state file."
  - "internal/battencli overview: settle whether the orch poll applies the run lines' two-strike tolerance, loads the soft and hard thresholds once at start, and shows `watcher down` only once the watcher has exited."
  - "Settle whether the diff leaves `lyx orch status`, the watcher, the orch state fields and the orch config unchanged, as the plan's Shared Decisions require."
  - "Test economy: settle whether the new tests cover PeekState and the orch line's phase, cycle count and watcher-down cases without duplicating existing orchengine or battencli tests, and whether any added behavior is left uncovered."
---

Round 1 focus set from the plan overview and card index; the plan's two cards map to the first two entries.
