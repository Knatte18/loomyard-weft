---
round: 3
focus:
  - "Card 3 (github per-request deadlines): does any production timeout that a test shortens stay a const, a struct field or a parameter, never a package-level var, as PATTERN-test-speed's Time-is-injectable rule requires, and does the page loop derive a fresh per-page context deadline?"
  - "Card 3's held-page test row: does its client carry githubclient's pause-gating transport (not the plain client from the selfreportengine test helper), so page 2 reaches the pause gate and the gate's sleep is asserted to run? Is its timing margin wide enough to survive -race under a loaded gate slot?"
  - "Card 4 (shuttle conditioned send): does the conditioned send path avoid the paste path's pending-send retryLanded shortcut, so a line never typed is never reported as landed? Does a test row run over an engine that implements text pasting, the path production's engine takes?"
  - "Card 5 (orch conditional notice): once delivery reads the orch's session state, does every comment and doc.go paragraph in orchengine that says no watcher decision depends on the state get updated by the card?"
  - "Card 6 (orch watch binary re-exec): does each refusal or guard branch in the safe-restart check name an input a test row can reach through the watcher's run loop, given that the tick itself loads state and fails first on an unreadable state file?"
---

Each entry names a site and a question; none caps severity or pre-states a result.
