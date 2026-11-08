---
round: 2
ledger:
  - key: sampler-plan-dir-untested
    rounds: [1]
    status: resolved
  - key: overview-regression-rows-omit-sampler
    rounds: [1]
    status: resolved
---

Round 1 raised two findings: an untested plan directory for the sampler's Drift expect agent, and a regression-rows section in `docs/overview.md` that omitted the sampler's scope gate and ordering.
Round 2 found nothing, so both are resolved.
