# Support log — orch-color-on-reexec

## Interview

A darn run holds no interview; the board entry is the whole statement of the change.

## Rejected alternatives

- Keep the pending mark in memory and pass it across the re-exec through an environment variable: `selfexec` re-execs generically, and the persisted state already survives every restart, including a crash.
- Key the record on the strand alone: an adopt relaunch always gets a new strand today, but `LaunchedAt` is what the board entry names and costs nothing to compare.
- Record the pair before typing: a held or failed typing would then never be retried.

## Review rounds

_No rounds yet._

## Question ledger

- Which time is the "session start"? Self-pick: `State.LaunchedAt`, set by `ResetForFreshLaunch` for every launched or adopted session. Resolved.
- Does the reload color step record the pair too? Self-pick: yes, every successful color typing records it. Resolved.
