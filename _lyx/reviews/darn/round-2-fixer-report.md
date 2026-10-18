# Round 2 fixer-report

## Changes

- F1 (MEDIUM): deleted the "A flaky creel test is fixed." bullet and its two continuation lines from `_lyx/landing/summary.md`, as proposed.
  The `_lyx` file is edited in the working tree only and not committed, because the round forbids git commands against `_lyx` paths.
- F2 (LOW): fixed with the first alternative, as proposed, in commit "PATTERN: fix F2".
  Restored the original text of `PATTERN-durable-vs-ephemeral-state`, `PATTERN-config-strictness` and `PATTERN-push-both-sides`.
  Shortened the `PATTERN-shed-recipe-registry` line to stay under the headroom line: "Adding a producer: ... behind `Lookup`, bar `Implementer`'s; no `init()`/`Register`/`lyxcwd`."
  `lyx loom lint-pattern` is clean and `lyx gate test ./internal/pattern` is green.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal, LOW: the shortened registry line drops `Names` and the word "shed".
  Disposition: disputed, ground `every fix worse`.
  Restoring `Names` and "shed" costs 11 bytes, and the overview is at its headroom line (`internal/pattern/check.go` `HeadroomBytes`).
  The only other cut is rewording an unrelated entry, which is the defect F2 names.

## Disputed

- fix-proposal LOW (registry line drops `Names` and "shed"), ground `every fix worse`: see Self-check.

## Deferred

Nothing deferred.
