---
round: 2
exclude_lenses: []
focus:
  - "Oversized single message: `internal/creelengine/verbs.go` `ClaimAtBoundary` hands out the first message whatever its size and `internal/creelengine/pool.go` `maxBodyBytes` is 64 KiB; check that decision-record.md states how one message larger than a hook field's 10,000-character cap is delivered, that the Constraints bound on that cap matches the rule chosen, and that an acceptance criterion covers it."
  - "Continuation count: check that decision-record.md orders the count reset against the cap check, states from evidence whether a wake resets Claude Code's own consecutive-continuation count, and that successive idle wakes with no mid-turn claim neither reach the cap nor strand the mail of a turn a typed bell started."
  - "Daemon restart: `internal/creelengine/server.go` exits the daemon on every binary replacement; check that decision-record.md states what a restarted daemon holds for each strand's count and woken-set marks, and that no chain's blocks can exceed creel's cap or lose journaled-read mail."
  - "Idle reading: `internal/creelengine/seam.go` and `internal/creelcli/delivery.go` `Idleness` answer a running turn, an active operator and a draft; check that the ring condition and the typed bell each use a reading the seam can compute, and that a draft holds a typed bell."
  - "Bell scope: shuttle sets `Provider` on every strand it launches (`internal/reedengine/state.go`) and a pre-deploy session's Stop hook only records (`internal/shuttleengine/claudeengine/settings.go`); check that no session whose Stop hook cannot deliver is belled, and that Open risks agrees."
---

Every entry names a code site and the question to settle; none restates a verdict.
