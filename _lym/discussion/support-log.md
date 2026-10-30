# Support log — dobby-land-notice-needs-flag

## Interview

A darn run holds no interview.
The writer asked the parent whether dobby should gate the can-land notice on the flag (A) or land the beam itself once flagged (B).
The parent took the question to the operator and asked for the parts both options share to be built meanwhile.

## Rejected alternatives

- The flag as a board field: the operator placed it in the beam's `_lym`, beside the landing's other information.
- The record under `.lym/beam/`, beside the land request: never-tracked state would be lost on a machine change, and the operator asked for `_lym`.
- A record with no slug: a pair forked from a flagged beam inherits its weft content and would read as flagged.
- The ready check before the pawl-run check in `lym beam land`: the run and owner refusals name ways forward that must be taken before the flag matters.
- A flag check only in the verb, not in Beam-Await's re-check: a land request is only a file, and the re-check re-runs every check of the verb.

## Review rounds

_No rounds yet._

## Question ledger

- Q1, open with the parent: (A) notice gate or (B) dobby lands the beam itself. Self-pick (A) as the entry's title states; see D6.
- Q2, self-picked: whether a verb clears the flag. No; the entry does not ask for one.
- Q3, self-picked: whether `lym beam ready` checks owners. No; it needs the beam's pair on this machine, and the landing's own owner checks still apply.
