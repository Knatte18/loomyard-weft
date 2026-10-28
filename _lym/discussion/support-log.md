# Support log — async-push-race

## Interview

A darn run holds no interview; the writer read the board entry and the push code and made every call itself.

## Rejected alternatives

- Serialising every push of one branch under one lock: the race's exact path is not pinned, and a lock change reaches every push caller, while a re-check before reporting covers any racing pair.
- Reclassifying a persisting rejection with origin contained in the local tip as `other`: not asked for, and it would change the class an existing hook-decline test pins.
- Stamping a machine name on records commits so the notice can name another machine: a new commit format for one notice line; the notice instead names no machine.
- One more retry with no backoff: the code side already retried once and the race still surfaced, so attempts are spaced by the push backoff.

## Review rounds

_No rounds yet._

## Question ledger

- Which sides get the re-check? Self-pick: all three that `pushClassified` pushes, since the seen notices were on code and records and Bolt pushes race the same way.
- What does a records attempt run? Self-pick: the side's existing replay retry, since a fetch failure inside it left origin's state unknown.
- Does a notice ever name another machine? Self-pick: never, since no commit carries a machine name.
