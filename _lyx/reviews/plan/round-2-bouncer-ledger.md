---
round: 2
ledger:
  - key: exit-hook-outside-d4-sites
    rounds: [1]
    status: resolved
  - key: reconcile-seeding-untested
    rounds: [1]
    status: resolved
  - key: card3-impactsummary-misses-fabric-clone
    rounds: [1]
    status: resolved
  - key: never-armed-case-missing-absent-config
    rounds: [1]
    status: resolved
  - key: card4-tests-omit-sink-redirect
    rounds: [1]
    status: resolved
  - key: stale-notifyexit-comment-in-sink-go
    rounds: [1]
    status: resolved
  - key: loggerconfig-float-values-truncate-not-reject
    rounds: [1, 2]
    status: resolved
  - key: card5-logger-yaml-fixtures-omit-second-key
    rounds: [1, 2]
    status: resolved
  - key: card2-rejection-tests-pass-on-missing-key
    rounds: [2]
    status: resolved
---

Round 1 reviewed the five-card plan and resolved six findings.
The review's findings covered the exit-hook site, reconcile seeding, card 3's ImpactSummary, the never-armed absent-config case, card 4's redirects, and a stale comment.
The judge blocked on card 2's float rejection.
yaml.v3 v3.0.1 truncates a float into an `int` field, so the stated mechanism did not carry D5.
The judge also noted, without blocking, that card 5's fixtures must carry both template keys.

Round 2's review raised both of those issues as F1 and F2.
It added F3: card 2's rejection tests could pass on `configengine`'s missing-key error.
The fixer resolved all three.
Card 2 now requires a `!!int`-tagged scalar per key and names float fixtures.
Every fixture carries both keys, and each rejection case changes one key and asserts the error names it.
The judge's spot-checks of cards 4 and 5 against the code found nothing new, and the plan is approved with no open entries.
