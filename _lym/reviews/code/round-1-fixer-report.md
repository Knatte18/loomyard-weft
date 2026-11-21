---
dispositions:
  - id: F1
    disposition: fixed
---

## Changes

- F1: in `TestReap` (`internal/hardenshed/reap_test.go`), the link-target subtest gained a process naming the written root by `Argv` with a `Cwd` outside both roots, plus an unrelated process; `want` is now both matching pids.
  A new subtest, `EndsAnOrphanUnderTheWrittenRootWhenTheRootDoesNotResolve`, reaps a root that does not exist and ends only the process under the written root.
  Commit d2151726, fixed as proposed.
- Self-check fix: the extended subtest is renamed `EndsAnOrphanNamingTheRootByItsLinkTargetOrItsWrittenPath`, since its old name understated what it asserts.
  The gate for `./internal/hardenshed` passed before the rename; the rename is committed after it.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, the subtest name no longer covers the written-root assertion; fixed as proposed by the rename.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
