---
round: 1
ledger:
  - key: card15-batten-rename-missing-users
    rounds: [1]
    status: resolved
  - key: card12-publish-verify-test-unedited
    rounds: [1]
    status: resolved
  - key: card10-shedbuild-fixture-missing-gate-and-rework-deps
    rounds: [1]
    status: resolved
  - key: card11-loomrecipe-landing-deps-missing-gate-fields
    rounds: [1]
    status: resolved
  - key: card16-stale-publish-awaiting-doc-statements
    rounds: [1]
    status: resolved
  - key: card6-runlevel-test-not-finished-run
    rounds: [1]
    status: resolved
  - key: card9-round-enumeration-unstated
    rounds: [1]
    status: resolved
  - key: card9-crash-after-clear-rejection-wedges
    rounds: [1]
    status: resolved
  - key: card12-impactsummary-restates-intent
    rounds: [1]
    status: resolved
  - key: card1-impactsummary-caller-miscount
    rounds: [1]
    status: resolved
---

Round 1 raised ten findings.
Three were BLOCKING, all build or test breaks from under-scoped Edit lists: the batten seam rename (card 15), the Publish verify tests (card 12) and the shedbuild every-engine fixture (card 10).
Three were MEDIUM: the loomrecipe landing fixture (card 11), four stale doc statements (card 16) and a mid-run rather than finished-run Webster resume test (card 6).
The rest were LOW or NIT: card 9's round-enumeration source and post-clear crash window, and the ImpactSummaries on cards 12 and 1.
The fixer addressed all ten before judgment, and a grep of the working tree confirmed each fix.
Round 1 is approved with nothing open.
