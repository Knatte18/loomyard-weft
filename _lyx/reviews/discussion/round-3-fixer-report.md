# Fixer report — discussion round 3

## Fixed

- **F1 (MEDIUM, design): move destinations are not journaled as created names.**
  Confirmed against `contracts/specs/loom-plan-spec.md`, which names card files `NN-<card-slug>.md`, so every move lands on a name that did not exist before the call.
  D2 now defines the journal's created names as every name that did not exist before the call, move destinations as well as the new card's file.
  The reversal sentence and `restore-renumber`'s restore bullet now remove every journaled created name, move destinations included.
  The injected-failure acceptance criterion now requires that no moved card's new-number file is left beside its old one.
- **F2 (MEDIUM, design): a failed journal write and an undecodable journal have no outcome.**
  D2 now writes the journal to a fixed temporary name and renames it into place, so a journal on disk is always complete.
  A failed write or rename removes the temporary file and joins the grouped raw-I/O row with the plan directory untouched, and a temporary file left by a kill is ignored and overwritten.
  `restore-renumber` gains a bound: a journal that does not decode is refused, writing nothing, as its own correctness-halt row.
  Its way forward is deleting the named journal file (a plain `rm <path>`, which `PATTERN-no-denied-recovery` keeps valid) and running `lyx loom validate-plan` (`internal/loomcli/validate.go`), whose findings name any inconsistent card.
  The acceptance criteria now cover a failure injected at the journal's write, no temporary journal left behind, and the undecodable-journal refusal.
  The support log's question ledger records this as Q11.
- **F3 (LOW, consistency): the widened raw-I/O row omits the journal and overclaims "nothing is mutated".**
  Scope, D2 and the acceptance criterion now widen the row to name the renumber journal too.
  `restore-renumber`'s failure partway through its writes is now its own transient row, with the way forward "re-run `lyx loom restore-renumber`; the journal is kept", and an acceptance criterion reaches it.

## Disputed

None.

## Deferred

None.
