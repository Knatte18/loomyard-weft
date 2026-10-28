---
round: 1
focus:
  - "internal/shuttleengine/claudeengine/usagelimit.go: does the `resets <time> (<zone>)` parse return the first instant after the entry's own timestamp in the named zone, for times with and without minutes, with a month and day, across midnight and DST, and read zero for an unknown zone or a missing clause?"
  - "internal/battenshed/notice.go: does the usage-limit episode key stay sent while any run of the child reads limited, end when none does, and send a second notice for a new limit; is the api-error notice and its revive never sent for a usage-limit turn end, while state change, dead driver and plain API error still notice?"
  - "internal/orchengine/usagelimit.go, watcher.go, state.go: does the watcher hold the orch strand's deliveries every tick, start no cycle while limited, release the hold and clear the record on a non-limit turn end, wake at reset plus one minute or 30 minutes after the read when no reset time is readable, and move the next wake 30 minutes on until the orch answers?"
  - "internal/orchengine/state.go: are the new orch state fields optional so an old record reads with no limit (PATTERN-tolerant-records)?"
  - "contracts/stencils and internal/orchengine/prompt.go: is the wake line read through `stencilstore` at call time with no prompt text in Go, and does the stencil name the reset and tell the orch to run `lym tender revive` and check every run?"
  - "internal/shuttleengine/activity.go and session.go: does the `UsageLimit` value on `TurnEndActivity` and `AgentActivity` follow only the newest main-chain turn end, and is the message parsing kept inside `claudeengine` (PATTERN-shuttle-provider-seam)?"
  - "Docs and tests: do the touched packages' `doc.go` files describe the new behavior, and do the new tests use injected clocks and cover each acceptance criterion without duplicating existing coverage?"
---

Focus follows the decision record's decisions D1 to D5 and its acceptance criteria.
