---
round: 1
focus:
  - "Follow-state read in internal/reedengine: does every decision treat only an explicit `off` as off, with unset reading on and a failed read counting as off for that one decision, logged at Warn, as the plan's Shared Decisions say?"
  - "Write and select failures: in every path except `lyx reed follow`, is a failed option write or failed select logged and never propagated, while `lyx reed follow` returns its own error envelope?"
  - "`followTargetWindow` versus `relevantWindow`: does the told-geometry rule read from window options yield the same window as the state-holding rule for the same facts?"
  - "`lyx loom start` entries: does every start entry call `FollowRun`, and does `lyx loom resume` not?"
  - "Manual window-change bindings and the follow button in the tmux config: do the bindings write follow off only on manual changes, not on changes lyx makes itself, and does the button toggle correctly in both directions and run `lyx reed follow` with the geometry it needs?"
  - "Test economy: do the new tests duplicate coverage across the engine, cli and loom layers, and does each added behavior have coverage?"
  - "Docs in the same commit: do the changed packages' `doc.go` files and `PATTERN.md`, where a new invariant arrived, describe follow and the new verb?"
---

Seed focus drawn from the plan's Shared Decisions and card index.
