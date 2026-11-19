---
round: 2
ledger:
  - key: card8-pattern-line-narrows-rule
    rounds: [1]
    status: resolved
  - key: hardenshed-fabric-side-tokens
    rounds: [1]
    status: resolved
  - key: hardencli-package-naming-deviation-unlisted
    rounds: [1]
    status: resolved
  - key: hub-gate-key-values-dropped-unrecorded
    rounds: [1]
    status: resolved
  - key: liverd-absent-report-reads-green
    rounds: [1]
    status: resolved
  - key: refused-live-call-reads-as-failed-run
    rounds: [1]
    status: resolved
  - key: gateslot-doc-and-test-speed-tables-stale
    rounds: [1]
    status: resolved
  - key: new-test-packages-lack-testmain
    rounds: [1]
    status: resolved
  - key: preflight-multi-tier-copies-key
    rounds: [1]
    status: resolved
  - key: reap-orphan-kill-no-caller-spare
    rounds: [1]
    status: resolved
  - key: efforts-refusal-no-way-forward
    rounds: [1]
    status: resolved
  - key: mergeresolve-treadle-comment-missed
    rounds: [1]
    status: resolved
  - key: runproc-doc-names-one-endprocess-user
    rounds: [1]
    status: resolved
  - key: config-layers-strict-callers-omit-hardenshed
    rounds: [1]
    status: resolved
  - key: board-recipe-enumerations-omit-hardener
    rounds: [1]
    status: resolved
  - key: hardenlive-run-reads-envelope-gate-never-prints
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1's fifteen keys are all resolved in round 2's review; none recurs.
Round 2 raises one new key, F16, `hardenlive-run-reads-envelope-gate-never-prints`:
card 14's `hardenLiveRun` expects a JSON envelope from `lym gate test --wait`, but only a refusal prints one on stdout.
A passing or failing run reports on stderr and by exit code, so every ordinary live run would error with no report.
The proposed fix changes the `hardenLiveRun` signature and contract in card 14 and the error sentence in card 6, so it is `design-wrong` and needs a fresh review.
