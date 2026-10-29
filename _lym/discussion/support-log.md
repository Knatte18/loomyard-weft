# Support log — webster-recover-batch-flake

## Interview

A darn run holds no interview.

## Rejected alternatives

- Building a fresh setup-and-check pair per row for the reused pairs: the rows composing `atStartSetup` would each need their own pair declared ahead of the table, which multiplies declarations without changing what is checked.
- Comparing the record by value instead of by pointer: it drops the check that a refused spawn did not replace the record.
- A map from fixture to record under a mutex: more machinery than a fixture field for the same per-subtest scope.

## Review rounds

_No rounds yet._

## Question ledger

- Resolved (self-pick): where the per-subtest record lives — on `recoverFixture`, built per subtest (D1).
- Resolved (self-pick): whether to also rework the single-row closure variables `scratchPath` and `longBesidePathBase` — no, one row writes and reads each, so they share nothing.
