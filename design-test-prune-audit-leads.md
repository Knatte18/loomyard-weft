# Act on the first test-time audit: reedengine wait, three over-grown test packages

`test-prune-audit-leads` · Task · enhancement, reed, board, batten · running · Code-Burler-Fix

## Seen

The first recurring test-time audit, `docs/benchmarks/test-suite-timing.md` section `test-time-audit: first recurring audit` (2026-10-28), found one slow tmux-tier package and three packages whose test count grew far beyond their code.

## Work

1. `internal/reedengine`, `tier2`: 2 tests take 40.00 s together, a round figure that reads as a fixed duration waited out.
   Find the wait and make it injectable under `PATTERN-test-speed`'s "Time is injectable", so the tests run in a fraction of that and still assert the same behavior.
2. Prune under `PATTERN-test-speed`'s "Test economy" in the three packages the audit named, each grown well beyond its changed code since 2026-10-07:
   - `internal/shell`: 11 to 65 top-level tests.
   - `internal/boardengine/boardtest`: 3 to 68.
   - `internal/battenrecipe`: 8 to 57.
   Read each package's tests against the behavior its code has, and fold or remove tests that assert nothing another test does not, into table rows where a table fits.
   No behavior loses its coverage; a test that pins a distinct behavior stays.
   Part of a count rise may be tests the older count missed because of its tags; a package whose tests each pin distinct behavior is left as it is, and the change description says so.

The change description states, per package, the test count before and after and why each removed or folded test was redundant.
