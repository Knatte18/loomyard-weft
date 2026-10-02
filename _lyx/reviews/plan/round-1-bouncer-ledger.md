---
round: 1
ledger:
  - key: card2-misses-webster-config-test
    rounds: [1]
    status: resolved
  - key: no-unparseable-yaml-refusal-test
    rounds: [1]
    status: resolved
  - key: card2-omits-presentcommentsonly-flip
    rounds: [1]
    status: resolved
  - key: tracked-skip-test-misses-prime
    rounds: [1]
    status: resolved
  - key: spawn-test-delete-drops-degrade-case
    rounds: [1]
    status: resolved
  - key: unless-name-flag-interaction-unstated
    rounds: [1]
    status: resolved
  - key: stale-load-strict-wiring-comments
    rounds: [1]
    status: resolved
  - key: backstop-description-narrower-than-missingkeys
    rounds: [1]
    status: resolved
---

Round 1 is the first round, with no prior ledger.
The review raised eight findings against cards 1, 2, 5 and 6.
The cards' targets and Intents now carry every fix, so all eight are resolved:

- **Card 2's test blast radius:** it adds the webster config test, the comments-only flip and the `wiring_test.go` comments.
- **Parse-error coverage:** cards 1 and 2 now test the unparseable-YAML refusal.
- **Card 6's tracked-skip test:** it now covers the prime, and the degrade-path test survives the `spawn_test.go` deletion.
- **Card 5's `--unless-name`:** it now states how the flag interacts with `--if-absent` and `--name`.
- **Card 2's backstop wording:** it now matches what `MissingKeys` reports.

Nothing is open.
