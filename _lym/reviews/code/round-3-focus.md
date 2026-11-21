---
round: 3
focus:
  - "internal/driftengine: after the diff, does any production code still read a stored old-Drift scope record, meaning a `partial` scope, a stored skip list, or the old compare, waiver and acceptance fields, in baseline.go recordScope, check.go, verdict.go or records.go; and does internal/driftengine/doc.go describe only behaviour the code still has?"
  - "internal/batcher/config.go retiredKeyError, LayerDefault arm: does each command it names remove the retired key when run, judged against what `lym fabric reconcile` and `lym config reconcile --layer default --apply` do with unknown keys, and do internal/batcher/doc.go and the config_test.go rows match the wording?"
  - "internal/configsync ReconcileHubWideAt, seed-only hub files: does a test seed a hub file from a prime copy while a template base exists and assert a template entry the prime copy lacks is added and listed in the result?"
---

Round 3 re-checks the three questions above against the code as it stands.
