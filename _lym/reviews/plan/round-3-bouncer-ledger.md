---
round: 3
ledger:
  - key: wire-seam-tests-nested-pullrequest-deps
    rounds: [1]
    status: resolved
  - key: wait-never-rereads-continue-record
    rounds: [1]
    status: resolved
  - key: d9-pause-replaced-without-decision
    rounds: [1]
    status: resolved
  - key: closed-pr-with-continue-skips-checks
    rounds: [1]
    status: resolved
  - key: resolver-github-host-test-undefined
    rounds: [1]
    status: resolved
  - key: continue-poll-breaks-once-a-minute-bound
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: shuttle-empty-outputfiles-precondition-falsified
    rounds: [1]
    status: resolved
  - key: stencil-card-names-wrong-symbol
    rounds: [1]
    status: resolved
  - key: falsified-doc-text-unlisted
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: prreview-gate-key-beyond-d3
    rounds: [1]
    status: resolved
  - key: pr-guard-misses-issue-endpoint-close
    rounds: [1]
    status: resolved
  - key: publish-deny-bound-under-reports
    rounds: [1]
    status: resolved
  - key: landingshed-import-allowlist-unedited
    rounds: [2, 3]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: pr-guard-rejects-envkit-stub
    rounds: [2]
    status: resolved
  - key: closed-for-continue-verdict-unreachable
    rounds: [2]
    status: resolved
  - key: gate-mail-reason-names-no-tier
    rounds: [2]
    status: resolved
---

Rounds 1 and 2 raised twelve and six findings; round 3 raises three, and the BLOCKING count fell from three to one.
Three keys remain open.
The landingshed allowlist gap came back in narrower form: card 14 lists the directive imports but not `internal/stencil`.
The poll-cadence key moved from the continue-flag poll to the entry read that card 11's step 6 and card 13's first wake both make; its fix hands the read status to `wait`, which changes an interface, so it is `design-wrong`.
The doc-text key now concerns two recipe header comments that say PR-Review never returns Stuck; the fix names them and is mechanical.
The round-2 keys the round-3 review no longer carries are resolved.
