# lym loom pr-findings fails: NewParentAlign: Deps.OpenFabric must not be nil

`pr-findings-open-fabric-nil` · Task · bug, loom, landing · running · Drift-Expect

Imported from [issue #187](https://github.com/Knatte18/loomyard/issues/187).

Seen: during a run's PR-Review row, lym loom pr-findings failed with: NewParentAlign: Deps.OpenFabric must not be nil.

Workaround: the PR findings were sent to the PR-Review agent by creel mail instead.

Expected: pr-findings builds its parent-align dependencies itself and records the findings.