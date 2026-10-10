# Card 8 — prerun-seed-pull-first

**Edit:**
- `cmd/lyx#seedStencils`
- `cmd/lyx#seedStencilsAt` -> `func seedStencilsAt(hub, worktree string, mode stencilstore.Mode, running buildvcs.Identity)`
- `cmd/lyx#seedSubtree`
- `cmd/lyx/stencilseed_integration_test.go`

**Prosa:**
- `//docs/overview.md`

**Uses:**
- `plan:internal/stencilstore#WritesDue`
- `internal/stencilstore#ModeFor`
- `plan:internal/fabricengine#Bolt.PullThenCommitWritten`
- `plan:internal/fabricengine#SeedCommitMessage`
- `plan:internal/fabricengine#BinaryLabel`
- `internal/fabricengine#StencilSource`
- `internal/fabricengine#NewBolt`

**Intent:** The root pre-run under D1 to D4.
`seedStencils` computes the mode through `ModeFor` from `buildinfo.IsDev()`, `buildinfo.IsProduction()` and `buildvcs.Running().Clean()` and hands the mode and identity to `seedStencilsAt`, which takes them as parameters so the integration test can drive a production mode and a chosen identity without a stamped test binary.
`seedSubtree` first asks `WritesDue`; with nothing due it returns having run no git and touched no network, the common case.
With a write due it calls `NewBolt(BoardDir(hub)).PullThenCommitWritten` with `SeedCommitMessage(label, BinaryLabel())` and a `write` that runs `Reconcile` against the pulled copy and returns its written paths prefixed with `subtreeRel`; a skipped result is logged once more at Info with the subtree, since the Warn already fired inside Bolt, and the pass stays best-effort.
`CommitSeededStencils` is no longer called from here.
`docs/overview.md`'s stencil row describes the forward-only refresh by build ancestry or commit time, the production-only refresh with the unstamped and dev warnings, the writer keys in the banner, the pull-first seed commit, the labelled subject and the seed-commit drop at push.
`TestStencilSeeding_HubScenario` passes `stencilstore.ModeProduction` and a fixed identity and gains serial steps: the first seed's commit subject matches `lyx: seed stencils (` with the label's revision, channel and executable; a second pass with nothing due succeeds with the board origin pointed at an unreachable path, proving no fetch ran; a deleted stencil with the origin still unreachable is not reseeded and the fetch Warn is captured; with the origin restored and a commit pushed to the records bare from a second clone, the reseed lands on top of the pulled copy; a seed commit made ahead of a moved upstream is dropped and regenerated; a board-style commit ahead of a moved upstream skips the write with the divergence Warn naming `git pull --rebase`.
The unstamped path is covered by `TestSeedStencils_NoOpAndEnvelopeUnderRoot` as today.

**ImpactSummary:** Re-signs one private helper with two callers in the same package, both listed; the pre-run's observable change is a seed commit's subject and the network reach on a due write, both covered by the scenario test.
