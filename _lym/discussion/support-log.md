# Support log — card-gate-misses-tier1

## Interview

A darn run holds no interview; the board entry and its Design section are the whole statement of the change.

## Rejected alternatives

- Run tier1 in the card gate: the board entry's Design rejects it, since a tier1 run per card costs more than the late find and the batch gate catches the breakage.
- Put the not-run tiers into the run record: no reader of the record decides on it, and the entry asks for the result the implementer reads.
- Print the line on every narrowed call: noise on the darn and fix gates, whose tier lists are the hub's chosen coverage.
- Print the line on a failing call too: a failure is not read as green, and the failed-runs line already closes it.

## Review rounds

_No rounds yet._

## Question ledger

- Self-pick: stderr line rather than an envelope, since a passing call prints no envelope. Resolved.
- Self-pick: manual tiers are named as not run, since they did not run. Resolved.
- Self-pick: only the card scope form prints the line. Resolved.
