# Support log — webster-tree-moved-fixer

## Interview

A darn run holds no interview.
The writer read the board entry, the Master stencil's plan-conflict rung, `begin-batch`'s conflict record, the shed Webster adapter's reroute claim and the `fix-verify` cold fixer, and settled the calls below alone.

## Rejected alternatives

- A fixer fork Master spawns in its own session: a fork inherits Master's whole context, so it is not small, and Master's rules bar it and its forks from the plan.
- Fixing the conflict in Go by rewriting renamed names automatically: Go cannot tell a rename the card should follow from a removal the card's work depends on, and the rename map is not recorded per batch.
- A new `plan_fixer` key in `webster.yaml`: no hub has asked for a separate knob, and the master role's model fits the job.
- Classifying mechanical findings by check name in Go (`glyph-not-found`, `path-missing`) before spawning: the same check name covers both a rename and a removed unit, so the agent's verdict decides instead.
- Having Master end stuck at once on a `plan_defect` cause: the first batch can meet a moved tree after a parent merge-in too, and the fixer's verdict covers a real defect.
- Listing the plan fixer's session in the run audit with an exemption for plan-directory writes: an exemption class in the audit weakens a correctness rule every other session relies on, for a write `rebaseline` already checks.

## Review rounds

_No rounds yet._

## Question ledger

- Q1: Fork or cold strand for the fixer? Self-pick: cold strand through a new verb (D1).
- Q2: Which model? Self-pick: the master role's choice, no new key (D2).
- Q3: Who decides a conflict is mechanical? Self-pick: the fixer's reply verdict (D4).
- Q4: How is the fixer bounded? Self-pick: once per finding set, twice per batch (D6).
- Q5: Does loom or shed change? Self-pick: no; a stuck run with the conflict record still reroutes (D5).
- Q6: Does a `plan_defect` cause go to the fixer? Self-pick: yes (auto-mode assumption).
