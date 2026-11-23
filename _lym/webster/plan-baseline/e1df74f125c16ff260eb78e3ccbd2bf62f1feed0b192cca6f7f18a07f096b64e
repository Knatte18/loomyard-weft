# Card 9 — runread-local-code-tip

**Edit:**
- `internal/pairteardown#runReader`
- `internal/pairteardown#hubRunReader`
- `internal/pairteardown#runReader.read`
- `internal/pairteardown#runFakes`
- `internal/pairteardown#runFakes.reader`
- `internal/pairteardown#TestReadRun`

**Uses:**
- `internal/fabricengine#HeadContains`
- `internal/pairteardown/doc.go`

**Intent:**
Let a run on its own machine act while its own pusher owes the code side of its last transition (D6).
A pair's records push lands before its code side, so origin names a `Warp-SHA` it lacks until the pusher's code push lands.

- `runReader` gains `localHoldsTip func(slug, tip string) (bool, error)`;
  `hubRunReader` fills it with `fabricengine.HeadContains` on the pair's code worktree.
- `runReader.read`, for a missing tip while the pair stands here, asks `localHoldsTip`:
  a local code branch holding the tip leaves `CodeTipMissing` and its fields unset, since the run's own pusher owes it.
  A tip the local branch lacks, and any missing tip while the pair stands elsewhere, set them as today,
  which `runfsm.CodeTipGrace` and the dobby's code-tip settle cover.
  A failed `localHoldsTip` is returned naming the read, like every other read.
- The owner dobby's code-tip settle then sees no `code_tip` verdict and pushes nothing;
  the dobby's pusher restart (card 8) carries the push.

`runFakes` and its `reader` gain the seam.
`TestReadRun` is tier0 and gains rows: a missing tip with the pair here whose branch holds it reads no `CodeTipMissing`;
the pair here whose branch lacks it reads `CodeTipMissing`; the pair elsewhere reads `CodeTipMissing` and never calls the seam;
a failing seam returns its error.

**ImpactSummary:** One new reader input and one branch in the run read; the act rule and its other readers are unchanged.
