---
round: 1
ledger:
  - key: d7-baseline-inside-batch-1
    rounds: [1]
    status: resolved
  - key: failures-set-and-tail-source
    rounds: [1]
    status: resolved
  - key: d3-master-stuck-recording
    rounds: [1]
    status: resolved
  - key: d2-rationale-stuck-summary-contract
    rounds: [1]
    status: resolved
  - key: outcome-yaml-after-demotion
    rounds: [1]
    status: resolved
  - key: d9-predicate-absent-or-not-run
    rounds: [1]
    status: resolved
  - key: stale-fork-log
    rounds: [1]
    status: resolved
  - key: auto-mode-assumptions-duplicate-decisions
    rounds: [1]
    status: resolved
  - key: plan-time-facts-in-d11-and-notes
    rounds: [1]
    status: resolved
  - key: d11-warning-per-verdict-ambiguity
    rounds: [1]
    status: resolved
---

Round 1 raised ten findings.
The blocking one was the D7 baseline: the first parent of batch 1's recorded head can sit inside batch 1 and excuse a regression.
The medium one was D10 listing only first-run failures, so a rerun-only failure had no entry or tail.
The rest were a narrower D3 statement, a misstated D2 rationale, an unstated `outcome.yaml` behavior on demotion, an underspecified D9 predicate, and a stale fork log, plus three leanness and wording nits.

The fixer resolved all ten in the same round.
The baseline now comes from the per-batch `StartSHA` webster already persists, with D8 covering its absence, and `failures` is the head set plus first-run-only identities.
A fresh pass found nothing new, so the pair is approved at round 1.
