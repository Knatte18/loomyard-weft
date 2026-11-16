---
round: 2
ledger:
  - key: printed-covering-list-not-the-judged-pick
    rounds: [1]
    status: resolved
  - key: candidate-cover-keeps-unconfirmed-kill
    rounds: [1]
    status: resolved
  - key: unbuildable-rebuild-loop-untested
    rounds: [1]
    status: resolved
  - key: guard-exclusion-untested
    rounds: [1]
    status: resolved
  - key: covering-pool-no-deterministic-test
    rounds: [1]
    status: resolved
  - key: testplanmutants-missing-keep-marker
    rounds: [1]
    status: resolved
  - key: sourcescan-memo-call-cycle
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: sourcescan-imported-selector-matches-methods
    rounds: [1]
    status: resolved
  - key: timeout-key-docs-omit-redundant
    rounds: [1]
    status: resolved
  - key: readme-old-redundant-definition
    rounds: [1]
    status: resolved
  - key: mutant-timeout-bound-untested
    rounds: [1]
    status: resolved
  - key: secondary-check-paths-without-row
    rounds: [1]
    status: resolved
  - key: redundant-test-waits-real-timeouts
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
---

Round 1 raised twelve findings; round 2 re-raises none of them except in one place.
F2 of round 2 returns to the call-cycle memo (`sourcescan-memo-call-cycle`): the test samples random walk orders, so it stays open at LOW.
F1 of round 2 is new: `TestGateRedundant` runs the verb through `Command()`, whose `secondUnit` is `time.Second`, so it waits out real 10 s mutant timeouts, about 41 s per run.
The fix drives the subtest through `runRedundant` with a short `secondUnit`, and it also makes the fixture pin that the timeout floor binds below the test bound.
That folds in the remainder of the round-1 key `mutant-timeout-bound-untested`, which is resolved here.
