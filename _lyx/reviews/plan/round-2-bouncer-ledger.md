---
round: 2
ledger:
  - key: wiring-reflect-deletes-value-and-identity-assertions
    rounds: [1]
    status: resolved
  - key: lyxbin-sync-once-over-tempdir-dangles
    rounds: [1]
    status: resolved
  - key: new-non-test-spawn-sites-trip-spawn-observability
    rounds: [1]
    status: resolved
  - key: target-lists-miss-callers-of-deleted-helpers
    rounds: [1, 2]
    status: resolved
  - key: kit-on-kit-exemption-unbounded-on-spawning
    rounds: [1]
    status: resolved
  - key: clone-remedy-string-wrong-owner
    rounds: [1]
    status: resolved
  - key: sentinels-impactsummary-no-blast-radius
    rounds: [1]
    status: resolved
  - key: kit-and-conversion-bundled-in-one-card
    rounds: [1]
    status: resolved
  - key: kit-cards-omit-d11-package-doc
    rounds: [1]
    status: resolved
  - key: race-only-assertions-never-run-under-race
    rounds: [1]
    status: resolved
  - key: mergeresolve-frictionengine-shuttle-fakes-unconverted
    rounds: [2]
    status: resolved
  - key: parentreview-embedder-missing-from-card-22
    rounds: [2]
    status: resolved
---

Round 1 raised ten findings against the generation-0 plan.
The fixer split it into 45 cards, and the judge confirmed nine fixes.
One class stayed open: `target-lists-miss-callers-of-deleted-helpers`, which recurred in cards 19 and 44.

Round 2 closed it.
Card 19 now edits every fabricengine caller of `newPortalLauncherTestLocation`, and card 44 every reedengine caller of `equalStringSlices`; a `grep -rlw` sweep at HEAD confirms both lists are complete.
Round 2 also settled the carried focus question on the `mergeresolve` and `frictionengine` `fakeShuttle` copies.
Neither package is cycle-forced, so the acceptance criterion requires their conversion.
Card 25's `MergeShuttle` now reproduces the richer mergeresolve copy, and card 29 converts both.
A nit on card 22 is fixed: it now names `smoke_parentreview_test.go`'s `parentNoticeEngine` as a dependent embedder.

Every entry is resolved, and the plan is approved at round 2.
