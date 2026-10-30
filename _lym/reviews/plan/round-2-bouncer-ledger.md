---
round: 2
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
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: shuttle-empty-outputfiles-precondition-falsified
    rounds: [1]
    status: resolved
  - key: stencil-card-names-wrong-symbol
    rounds: [1]
    status: resolved
  - key: falsified-doc-text-unlisted
    rounds: [1, 2]
    status: open
    class: scope
    severity: MEDIUM
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
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: pr-guard-rejects-envkit-stub
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: closed-for-continue-verdict-unreachable
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: gate-mail-reason-names-no-tier
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
---

Round 1 raised twelve findings; the round-2 review no longer carries ten of those keys, so they are resolved.
Two reappear in new form: the doc-text gap (README and a loomrecipe doc sentence still falsified) and the one-time immediate poll on Continue that departs from D9's once-a-minute bound.

Round 2 adds four keys.
Two BLOCKING: card 11 pulls `internal/pullrequest` and `internal/githubclient` into landingshed without editing its import allowlist, and card 16's new import guard rejects the envkit stub card 11 requires.
Two MEDIUM: the closed-and-`ClosedForContinue` Done verdict in `wait` is unreachable because `Call` deletes the record first, and the agent mail's gate reason names no tier.
The last two change behavior or add a helper, so they are `design-wrong`; the rest are local corrections the finding text fixes.
