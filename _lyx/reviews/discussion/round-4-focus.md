---
round: 4
focus:
  - "In `_lyx/discussion/decision-record.md` D16 and D17, settle whether the orch can tell a landed task from a still-running one after a dropped landing notice on a hub whose `landing.yaml` sets a non-empty `deploy` key, where the Land row runs the deploy and `origin/main` and `origin/prod` agree after every landing. Name the durable baseline each fallback check reads and whether it depends on the deploy key."
  - "In D17, settle whether the fallback that starts a ready task is tied to the same wake as any deploy-key-gated check, and whether a landing into a parent task branch rather than `main` is covered by the baseline D16 names."
  - "In D11 and the Scope bullet of `decision-record.md`, settle whether the audit's stated reach reads the same everywhere: which Warn sites are audited, which are exempt, and whether an exempt site is still given a verdict."
---

The first two entries follow the attack-surface question for the not-retried escape hatch: what it can skip and what bounds it.
