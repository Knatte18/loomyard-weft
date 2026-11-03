---
round: 1
focus:
  - "internal/fabricengine/pushlockholder.go: does the holder record's read, pid and age derivation, and liveness check handle a missing, empty, malformed or racing lock file without a wrong verdict or a panic?"
  - "Goal asks to clear a lock whose holder is dead: does the diff clear such a lock, and if it does not, is that omission a scope gap against the Goal's 'clear a lock whose holder is dead'?"
  - "internal/loomcli/landingdeps.go: does the retry with growing pauses terminate, honour context cancellation, retry only the busy-lock error and not other push failures, and still end in a blocked run with the holder named when the lock stays busy?"
  - "internal/fabricengine/pushanchored.go, pull.go, pendingflags.go, originreconcilemoves.go: does the busy-lock error's wrapping still reach callers that match on it, and is the holder text added at every site that returns it?"
  - "cmd/lym/destructiveguard_test.go and the allowlist entry: is the allowlisted removal limited to the holder record, and is the removal justified?"
  - "Test economy: do pushlockholder_test.go and landingdeps_test.go cover each added behavior (liveness, age, retry, exhaustion) once, without duplicating an existing test?"
  - "Comments and summary: do new doc comments follow the repository's conventions, and does any claim in _lym/landing/summary.md, if present, fail to match the diff?"
---

Seed set from the Goal in the decision record, which is the only answer key (no plan).
