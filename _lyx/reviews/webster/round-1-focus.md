---
round: 1
focus:
  - "internal/shuttle write path (cards 5-8): does every write, including /clear, /compact, /reload-plugins and /color, take the one guarded path with a single fresh readiness, activity and readable-box reading before the paste?"
  - "Shuttle's Enter after a paste (card 6) and the pending-send record (card 7): is Enter sent only when the box holds exactly the sent text, and does a retry after a possibly-left send type nothing twice, including after a crash between the paste and the record write?"
  - "Reed paste-submit (card 1) and the /rename watchdog (card 3): is the paste and Enter one tmux invocation using `set-buffer -b <name> --`, is the named buffer cleaned up, and does the rename sit behind the activity check?"
  - "Claude paste path (card 9): is the leading Escape gone from every engine path, and does the settle apply only where the shared decision settle-only-for-the-no-enter-variant says it does?"
  - "Held-write handling in the orch watcher (card 11), loom driver resume (cards 12-13) and batten (cards 14-15): is a held write or busy driver treated as 'later' everywhere, and is the resume report path reused only while the park marker content is unchanged?"
  - "Batten status halted flag (card 16) and the after-landing stencil (card 17): does the flag fire only for a halted batten over a done child, and does the stencil close batten as the plan states?"
  - "Test economy across the diff: which new tests duplicate coverage that an existing test already gives, and which added behavior has none?"
  - "Per-card mechanical checks: did each group's type-specific check (Rename, Delete, and the rest) run against that group's own targets?"
---

Seed focus drawn from the plan's card index and shared decisions.
The review range is defined in the rubric.
