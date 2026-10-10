# Card 8 — prerun-seed-pull-first

**Edit:**
- `cmd/lyx#seedStencils`
- `cmd/lyx#seedStencilsAt` -> `func seedStencilsAt(hub, worktree string, mode stencilstore.Mode, running buildvcs.Identity)`
- `cmd/lyx#seedSubtree` -> `func seedSubtree(baseDir, subtreeRel string, registry stencilstore.Registry, mode stencilstore.Mode, source stencilstore.Source, label string) (fabricengine.BoltWrite, bool)`
- `cmd/lyx/stencilseed_integration_test.go`

**Prosa:**
- `//docs/overview.md`

**Uses:**
- `internal/stencilstore#WritesDue`
- `internal/stencilstore#ModeFor`
- `internal/fabricengine#Bolt.PullThenCommitWritten`
- `internal/fabricengine#BoltWrite`
- `internal/fabricengine#SeedCommitMessage`
- `internal/fabricengine#BinaryLabel`
- `internal/fabricengine#StencilSource`
- `internal/fabricengine#NewBolt`
- `internal/fabricengine#NewMutations`

**Intent:** The root pre-run under D1 to D4, as one pass over both subtrees.
`seedStencils` computes the mode through `ModeFor` from `buildinfo.IsDev()`, `buildinfo.IsProduction()` and `buildvcs.Running().Clean()` and hands the mode and identity to `seedStencilsAt`, which takes them as parameters so the integration test can drive a production mode and a chosen identity without a stamped test binary.
`seedSubtree` no longer commits: it asks `WritesDue` for its subtree and returns whether a write is due beside a `BoltWrite` whose `Message` is `SeedCommitMessage(label, BinaryLabel())` and whose `Write` runs `Reconcile` against the pulled copy and returns its written paths prefixed with `subtreeRel`; a `WritesDue` error is logged at Warn and reports nothing due for that subtree.
`seedStencilsAt` builds both subtrees' writes first; with neither due it returns having run no git and touched no network, the common case.
With either due it makes one `NewBolt(BoardDir(hub)).PullThenCommitWritten` call carrying both writes, stencils then specs, and a local `NewMutations(hub)` record, since the pre-run has no envelope and a drop is logged inside Bolt, so one pull and one drop precede both writes and the specs write can never drop the stencils seed commit the same pass made; a subtree with nothing left due after the pull commits nothing.
A skipped result is logged once more at Info with the skip, since the Warn already fired inside Bolt, and the pass stays best-effort.
`CommitSeededStencils` is no longer called from here.
`docs/overview.md`'s stencil row describes the forward-only refresh by build ancestry or commit time, the production-only refresh with the unstamped and dev warnings, the writer keys in the banner, the pull-first seed commit, the labelled subject and the seed-commit drop at push.
`TestStencilSeeding_HubScenario` passes `stencilstore.ModeProduction` and a fixed identity and gains serial steps: the first seed, with both subtrees due, lands a `lyx: seed stencils (` and a `lyx: seed specs (` commit, both still on Bolt after the pass, each subject equal to `SeedCommitMessage(<subtree>, BinaryLabel())` computed in the test process, since the label names the running binary and never the fixed identity, which drives only the writer keys; a second pass with nothing due succeeds with the board origin pointed at an unreachable path, proving no fetch ran; a deleted stencil with the origin still unreachable is not reseeded and the fetch Warn is captured; with the origin restored and a commit pushed to the records bare from a second clone, the reseed lands on top of the pulled copy; a seed commit made ahead of a moved upstream is dropped and regenerated; a board-style commit ahead of a moved upstream skips the write with the divergence Warn naming `git pull --rebase`.
The unstamped path is covered by `TestSeedStencils_NoOpAndEnvelopeUnderRoot` as today.

**ImpactSummary:** Re-signs two private helpers whose callers, `seedStencils`, `seedStencilsAt` and the scenario test, are all on this card; the pre-run's observable change is a seed commit's subject and the network reach on a due write, both covered by the scenario test.
