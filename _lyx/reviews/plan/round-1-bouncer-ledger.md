---
round: 1
ledger:
  - key: card3-unarchived-test-missing-weft-branch-assert
    rounds: [1]
    status: resolved
  - key: card3-skippush-test-warp-side-unchecked
    rounds: [1]
    status: resolved
  - key: card3-impactsummary-caller-set
    rounds: [1]
    status: resolved
  - key: card3-warp-ff-pass-branch-tested-only-in-card5
    rounds: [1]
    status: resolved
  - key: card5-impactsummary-rollback-and-hook
    rounds: [1]
    status: resolved
  - key: card2-empty-lease-refusal-untested
    rounds: [1]
    status: resolved
  - key: card3-ancestor-helper-duplicates-isancestor
    rounds: [1]
    status: resolved
  - key: card2-stale-destroyremote-test-header
    rounds: [1]
    status: resolved
  - key: card3-wrong-followup-card-number
    rounds: [1]
    status: resolved
---

This is round 1, so there is no prior ledger.
The reviewer raised nine findings: one MEDIUM, six LOW and two NIT.
They were about Card 3's test post-conditions and ImpactSummary, where the warp fast-forward test sat, Card 5's ImpactSummary, a missing test for Card 2's empty-lease refusal, a helper that duplicated `gitrepo.IsAncestor`, a stale test-file header, and a wrong card cross-reference.
The fixer applied all nine before this judge pass, and each one was checked against the current plan files.
All nine are resolved.
The plan is approved at round 1.
