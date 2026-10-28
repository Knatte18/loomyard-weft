# Support log — usage-limit-wake

## Interview

A darn run holds no interview: the writer read the board entry and the code and made each call itself.

## Rejected alternatives

- A hub-wide usage-limit record that batten writes and the orch watcher reads: two writers of one limit and a new file, where the orch's own turn end already tells the watcher.
- Holding batten's notices inside batten until the reset: batten has no way to wake the orch, and the creel hold already exists for the watcher.
- Keeping the api-error revive for a usage limit: each revive typed into a limited session caused the repeat notices.
- Wake line as a Go constant: the Stencil Ownership Invariant puts prompt text in a stencil.

## Review rounds

_No rounds yet._

## Question ledger

- Where is a usage limit's reset time read? Self-picked: the Claude engine parses the limit message's `resets` clause; the classification alone decides that a limit stopped the turn.
- One notice per strand or per hub? Self-picked: per task run and limit, keyed by the reset time.
- Does `lym tender revive` need a change? Self-picked: no; it skips a busy strand Claude Code already resumed.
- What wakes the orch? Self-picked: the orch watcher injects a stencil line at reset plus one minute, then every 30 minutes until a turn end that is not a usage limit.
