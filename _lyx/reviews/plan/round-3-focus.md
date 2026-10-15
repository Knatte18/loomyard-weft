---
round: 3
focus:
  - "Card 9 (`_lyx/plan/09-creelengine-delivery.md`), delivery worker: for every state a worker can sit in for hours with nothing changing (receiver dormant with unread mail, receiver idle with its reminder inside the 5-minute to 1-hour backoff, busy, held, paused), does the wake interval double from the 1-second base to a named ceiling and reset on a stated event, as `PATTERN-daemon-wakeups` requires, and do the What-counts entry and the `TestScheduler_Reminders` waits row state the same rule?"
  - "Cards 7 and 9 (`_lyx/plan/07-creelengine-pool-verbs.md`, `_lyx/plan/09-creelengine-delivery.md`), interrupt and inject refusals: does exactly one card own the privilege, orch, pause and hold checks on `Store.Interrupt` and `Store.Inject` in `verbs.go`, with that card listing the Edit target, and does no card ask a later card to patch a file it does not list?"
  - "Card 7 (`_lyx/plan/07-creelengine-pool-verbs.md`) and card 9, orch unread set: does every verb that grows an orch strand's unread set (post, reply, forward, return) journal a `withheld` event, and do card 9's enumerations of those changes match the writers card 7 lists?"
  - "Card 7 privileged-inject Bound: does the stated TTL bound hold while the receiver is dormant or paused, and does the Bound name what the interrupt forms can do and what limits them?"
  - "Card 14 (`_lyx/plan/14-creelcli-verbs.md`) `TestCreelCLI`: does the card state how the claim's `LYX_STRAND_NAME` is set and cleared for strand and EXTERNAL rows, and whether the test may run in parallel, given the process-global environment and a platform where no peer is reported?"
  - "Cards 7, 9 and 10 test files over `creelfake`: is each test file that imports `creelfake` placed in a package that does not close an import cycle, and does `creelfake.Clock` record the durations that cards 9 and 10 read off it?"
---

Focus lists the sites where round 2's findings sat, restated without reference to the earlier review.
Each entry is a question to settle against the current plan text.
