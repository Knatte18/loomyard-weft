---
round: 2
ledger:
  - key: accepted-not-bound-to-head
    rounds: [1]
    status: resolved
  - key: pending-fix-review-resume-untested
    rounds: [1]
    status: resolved
  - key: spec-old-halt-rule
    rounds: [1]
    status: resolved
  - key: accept-path-coverage-gaps
    rounds: [1]
    status: resolved
  - key: tapestry-no-reroute-assertion
    rounds: [1]
    status: resolved
  - key: tests-model-finalize-reroute-cap
    rounds: [1]
    status: resolved
  - key: overview-circling-accept-bouncer-only
    rounds: [1]
    status: resolved
  - key: duplicate-resume-coverage-spent-cap
    rounds: [1]
    status: resolved
  - key: pending-resume-landing-unasserted
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised eight findings; round 2 does not re-raise any of them, and its focus verdicts confirm the HEAD-bound acceptance holds, so all eight are resolved.
Round 2 raised one new LOW finding, `pending-resume-landing-unasserted`: the pending-findings resume test checks only the error, not the landing.
Its fix is local test assertions the finding text already states.
