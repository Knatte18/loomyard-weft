# Support log — delete-order-own-edit

## Interview

A darn run holds no interview: the scope and every call come from the board entry and a reading of `internal/planglyph`.

## Rejected alternatives

- **Filter only in `LaterDeleteReferences`.**
  Rejected: the plan-gate path reaches the same `deleteOrderFindings`, and the two would then disagree on one plan.
- **Change `laterEditRegions` to exclude references that an own or earlier region also holds.**
  Rejected: `insideLaterEditRegion` builds on it for the caller-uncovered hand-off, where the meaning "a later card's region holds this line" must stay unchanged.
- **Extend the exemption to `laterImportFindings`.**
  Rejected for this task: the board entry names `deleteOrderFindings` only; the import arm is noted as out of scope.

## Review rounds

_No rounds yet._

## Question ledger

- Q: Does "a card before it" include done cards the caller does not pass? Self-pick: no; a done card's edit has run, so the tree the check reads already reflects it.
- Q: Should the deleting card's own Edit regions come from a separate list or the union with the later cards? Self-pick: the union, deduplicated by card number, since the plan-gate caller passes one list for both.
