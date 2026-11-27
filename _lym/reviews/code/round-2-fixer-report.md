---
dispositions:
  - id: F12
    disposition: fixed-differently
    reason: "The premise held, and the proposed changes were made except one: pattern/PATTERN-test-speed.md does not name three tests in place of the general phrase. The names gave a later .NET-building test no place, so the bullet keeps the category and says it covers every package, watched or not."
---

## Changes

Commits `c33d4c4e7` and `42a73a540`.

- `cmd/lym/csharpfixtures_integration_test.go`: `TestCSharpFixtures_CompileWithDotnet` carries `//lym:only upkeep`, and its doc comment says only the upkeep gate key runs it.
- `pattern/PATTERN-test-speed.md`, the `upkeep` bullet (fixed differently):
  it now reads that the dead-code guard and every test that builds a real .NET project name `upkeep`, whatever its package and whether or not it is watched.
  The proposed list of three test names was not kept.
  All three tests it listed carry the marker, so the category holds, and a list would go stale and leave a new .NET-building test without a rule.
- `pattern/PATTERN-test-speed.md`, the watched-guard exemption: it now covers a watched guard that reads a bounded set of files, as proposed.
- `pattern/PATTERN-test-speed.md`, the `//lym:watch` section, beyond the proposal:
  "A whole gate always runs it" now excepts a test whose `//lym:only` marker does not name the gate's key, the same false claim the finding removed from the test's doc comment.
- `_lym/landing/summary.md`: the `upkeep` bullet names the C# fixture compile.
  The watched-guard decision says such a guard reads a bounded set of files and that a scope-all gate runs it in full.

Checks after the changes: `lym gate test --tier 0 ./cmd/lym` passed (463 passed, 14 skipped).
`lym gate test --tier 1 ./cmd/lym -- -run '^TestCSharpFixtures_CompileWithDotnet$' -v` passed, and the test ran, not skipped.
No `--gate upkeep` call was run.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW (the summary's watched-guard decision reversed its reasoning): fixed as proposed.
- fix-proposal, LOW (the three names removed the rule for a new .NET-building test): fixed with own change.
  The category is stated without example names, which would go stale.
- fix-proposal, LOW (the summary's reversed reasoning, the same defect as the consistency finding): fixed with own change, using the consistency lens's wording.
- fix-affected-set: no findings.

## Disputed

## Deferred

## Rulings
