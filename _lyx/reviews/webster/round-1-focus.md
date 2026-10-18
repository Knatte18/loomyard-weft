---
round: 1
focus:
  - "Both declared PATTERN changes (`PATTERN-orch-pane-single-writer`, `PATTERN-durable-vs-ephemeral-state`): does the `PATTERN.md` and `pattern/` diff land each with the declared text, including the verbatim background sentence?"
  - "Card 8 (`drop-hub-notice-log`): is any reference to the removed notice log, its paths or its old names left in the diff's reach, and does `assert-no-callers` hold for each deleted symbol?"
  - "Cards 9 and 10 (orch watcher): are the notice queue and the idle probe before typing removed, and does every step reach the orch only through the creel seam, with delivery confirmed by creel's outcome as the plan states?"
  - "Card 11 (`orch-start-withdraws`): do both withdraw rounds and the watcher's retry of a failed round follow the plan, including the `<shortname>:orch` fallback name from the watcher-addresses-by-name decision?"
  - "Cards 4 and 5: does every Go client call carry `goCallTimeout`, and does the refusal code reach callers as typed, per the go-client-surface decision?"
  - "Card 6: does `orchcli.NotifyPrime` answer `(posted bool, err error)` as the notify-answer decision says, and does every notice caller in batten, loom, burler, webster, hibernate and shuttle use it?"
  - "Commit messages: does each `Pattern-Departure:` trailer name a known entry that forbids the card's way, and does it excuse only the difference it names?"
  - "Test economy: do new tests duplicate existing coverage, and does each added behavior have coverage?"
---

Focus covers the declared PATTERN changes, the removals whose leftovers are easy to miss, the new creel seams, and the departure and test-economy dimensions.
