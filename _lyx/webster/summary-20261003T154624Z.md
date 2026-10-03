# Pair lifecycle: teardown, remote branch cleanup, retiring driver

All 13 batches finished `done`, with no recovery needed.

- **01** (`a83236de8`): `Topology.RemoveRefusal` exposes `Remove`'s refusals as a read-only probe. `Remove` commits pending sibling records before the archive tag.
  - Deviation: `internal/fabricengine/remove_weftprobe_test.go`.
- **06** (`428408dbaa`): `reedengine.EndSessionByName` ends a session without an engine and tidies the tmux server after the last one.
- **07** (`51198eb38`): `Strand.Retiring` and `Engine.MarkRetiring`. `reed remove --detach` marks the strand, and the detached remover no-ops on a gone guid.
  - Deviation: `internal/reedcli/cli.go`, for a test seam.
- **02** (`80c0547d3`): `Remove` finishes a half-removed pair, reports `Steps`, `Finished` and `StrayPath`, and returns `ErrPairNotFound`.
  - Deviation: `internal/fabricengine/remove_refusal_remedy_integration_test.go`.
- **03** (`120843ca5`): `remove --remote` deletes the landed task branch on origin, leased, through a new remote-tip gate.
  - Deviations: `internal/fabricengine/export_test.go` and `internal/fabricengine/remove_finish_integration_test.go`.
- **09** (`efa94caa5`): new `internal/pairteardown` composite with `EndSession`, `RemovePair` and `Run`.
- **11** (`8a1ff1789`): batten's teardown closures call the composite's two phases, and not-found counts as done.
- **12** (`596c04d5c`): deleted `RemovePairBranch`, `PairBranchResult` and `PairSiblingRemnant`.
- **04** (`f8c716274`): `Topology.CleanupRemoteWarp` sweeps leftover task branches on origin.
  - Deviation: `internal/fabricengine/export_test.go`.
- **05** (`7b54607b8`): `cleanup --remote` fetches open PR heads and folds the sweep into its envelope.
- **08** (`4165a69a4`): `loom start` removes a retiring driver strand and spawns a fresh driver.
- **10** (`f1d278aab`): `lyx fabric remove` calls the composite, with the envelope and help updated.
- **13** (`f59c63cb7`): new Pair Teardown Invariant in `CONSTRAINTS.md` with its tripwire test, the module doc, and overview and package docs.

The plan-level integration suite reported `FAILED` at `f59c63cb7`. I did not triage it, and webster triages it after this session ends.


## Integration suite fix

A fix strand attempted the integration regression; result: `fixed`.
Pre-fix head: `f59c63cb75fcd42a54cd43a643fcbc8b50c16065`.

Fix commits:

- `d4d9f6b96daa562c203f4d64d07af9af07774f36`

Cleared regressions:

- `github.com/Knatte18/loomyard/internal/pairteardown`
