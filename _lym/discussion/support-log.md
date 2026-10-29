# Support log — test-prune-audit-leads

## Interview

A darn run holds no interview: the writer read the board entry, the audit section, the three packages' tests and a verbose gate run of `internal/reedengine`'s `tier2` tier, and made the calls itself.

## Rejected alternatives

- **Look for a fixed wait in reedengine and inject it, as the board entry words it.**
  No test in the package waits out a production duration; the slowest takes 11 s and the times sum to the package wall, so there is nothing to inject.
- **Add `t.Parallel` only to the tests that never touch the seam.**
  Parallel top-level tests start only after every serial one finishes, so the seam's tests, about 25 s, would still run serially before them.
- **Keep the package variable and guard it with a mutex.**
  The tests would still serialize on it, which is what the field removes.
- **Prune `internal/shell` by merging its per-method tables into one.**
  Each table pins a different method's output; one table over every method would cover the same behavior with no test removed for redundancy.
- **Fold `TestNew_CarriesShedPathsVerbatim` into the row-name test.**
  It pins a distinct behavior, that the returned shed carries the told paths, so it stays.

## Review rounds

_No rounds yet._

## Question ledger

- Where does reedengine's 40 s go? Resolved: a serial sum of 32 tests, no fixed wait (D1).
- Does the seam change touch production behavior? Resolved, self-pick: no; `New` sets `os.Executable` (D2).
- Do the three packages hold the counts the audit names? Resolved: no; 14, 3 and 9 tests (D4).
- Is the audit section corrected? Resolved, self-pick: yes, with a short note, its numbers kept (D6).
