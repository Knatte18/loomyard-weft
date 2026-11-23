---
round: 1
focus:
  - "internal/shedadapters/bouncerverify.go, bouncer.go: does the verifier close a BLOCKING ledger entry only when the fixer reported it fixed with a commit, and keep it open when the fix commit left the target unchanged or the verifier's ruling is missing, malformed or fails?"
  - "internal/shedadapters/bouncer.go blockingEscalationDetail and bouncerfiles.go: after verification, does the parent get asked only about entries still open, and is every failure path of the verifier (spawn error, unreadable ledger, failed ledger write) an honest escalation rather than a silent close or a silent skip?"
  - "Scope: the decision record's Goal also lists below-window-accept-blocking, budget-escalation-untested, code-auto-accept-flag-only and hardener-focus-wiring-untested; which of these does the diff land, and does it make any change the entry does not license?"
  - "contracts/recipes/*.yaml, contracts/specs/shed-recipe-spec.md, docs/overview.md, internal/shuttleparity/producers_test.go: do the recipe, spec and doc changes match the new verification step and the repository's rules for a new producer or stencil?"
  - "contracts/stencils/bouncer/bouncer-template-verify.md: does the verify stencil restrict the verifier to the named findings and fix commits, and does its output format match what parseVerification reads?"
  - "internal/shedadapters/bouncer_verify_test.go: do the tests assert independent expected values, cover the diff's added behavior (dispute, deferral, no fixer report, unchanged target), and avoid duplicating TestBouncer_AutoAccept coverage?"
---

Seed focus from the stat, commit message and decision record.
No prior round exists.
