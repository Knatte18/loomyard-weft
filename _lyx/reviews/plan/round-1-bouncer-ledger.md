---
round: 1
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
    rounds: [1]
    status: open
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
---

Round 1's review raised ten findings against the generation-0 plan, two of them blocking: the reflective wiring tests deleted value and identity assertions, and `lyxbin`'s `sync.Once` guidance left a dangling binary.
The fixer rewrote the plan to 45 cards, and the judge confirmed all ten fixes against HEAD.

One class stays open: `target-lists-miss-callers-of-deleted-helpers`.
Every card F4 named is now complete, but two cards it never named repeat the defect.
Card 19, a product of the kit/conversion split, deletes `newPortalLauncherTestLocation` without targeting its callers in fabricengine's `hubworkspace_test.go`, `launcher_content_test.go` and `origin_test.go`.
Card 44 deletes reedengine's `equalStringSlices` without targeting `strand_test.go` and `lifecycle_test.go`.

A focus question carries to round 2 without a ledger entry: whether the in-package `Shuttle` fakes in `mergeresolve` and `frictionengine` fall under the acceptance criterion's no-local-fake rule.
