# Fixer-report, round 1

## Changes

- F1: no change. The review's `fix` says to keep the code and leave recording the decision to the parent; fixed as proposed by leaving it in place.
- F2: fixed as proposed. `internal/gatecli/test_integration_test.go` gains a subtest of `TestGateTest_Scenario` that holds the pool's `drain.lock` with `lock.TryAcquireWriteLock` and asserts `lyx gate status` reports `draining: true`, releasing the lock on cleanup. Run under `lyx gate test --tier 1 ./internal/gatecli`: green.
- F3: fixed as proposed. The `--tier N` line in `testCmd`'s `Long` text (`internal/gatecli/cli.go`) now says the default is tier0 unless `--upto` is given.
  The first wording pushed `Long` to 2011 bytes, over the 2000-byte cap that `TestCLIForm_EveryCommand` enforces, so the gate failed; the shorter wording fits.
  The self-check found the twin sentence in `internal/gatecli/doc.go` and it got the same wording in a separate commit.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, `doc.go:8` still said "the default is tier0"; fixed as proposed.
- fix-proposal: LOW, same `doc.go:8` twin (duplicate of the fix-consistency finding); fixed as proposed. No finding against the F2 or F3 proposals themselves.
- fix-affected-set: LOW, same `doc.go:8` twin (duplicate); fixed as proposed.

## Disputed

None.

## Deferred

None.
