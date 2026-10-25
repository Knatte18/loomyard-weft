---
round: 2
ledger:
  - key: capture-job-kills-step-process
    rounds: [1]
    status: resolved
  - key: drift-verify-probe-after-mechanical-steps
    rounds: [1]
    status: resolved
  - key: drift-verify-step6-trigger-too-wide
    rounds: [1]
    status: resolved
  - key: card14-misses-routing-test-and-phase-prose
    rounds: [1]
    status: resolved
  - key: dirty-tree-way-forward-at-drift-await
    rounds: [1]
    status: resolved
  - key: drift-baseline-commits-capture-logs-midwrite
    rounds: [1]
    status: resolved
  - key: extends-on-rewritten-refs
    rounds: [1]
    status: resolved
  - key: runs-spent-refusal-unreached
    rounds: [1]
    status: resolved
  - key: accept-agent-attribution-by-running-status
    rounds: [1]
    status: resolved
  - key: startcapture-errors-no-way-forward
    rounds: [1]
    status: resolved
  - key: gatecli-doc-gate-key-example-missed
    rounds: [1]
    status: resolved
  - key: capture-command-two-audiences
    rounds: [1]
    status: resolved
  - key: step7-resume-reads-unkept-stuck-reason
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1 raised twelve findings, two of them BLOCKING design.
Round 2's review re-raised none of them, so all twelve are resolved.

Round 2 raised one new BLOCKING design finding, F1.
The drift-verify producer's step 7 tells a resume from the row's own dirty-tree or test-gate Stuck by that Stuck's reason text, which the run status does not keep at the resumed entry.
The proposed fix moves the discriminator onto a scope-record mark the row writes itself, with matching changes to the producer's step 10 and its test rows.
That fix changes a record contract, so it is `design-wrong` and needs a fresh review.
