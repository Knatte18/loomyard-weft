# bolt-and-store: Bolt and hub-store writes made correct and quiet

All three batches reported `done`.

## Batch 01 — buildvcs-commit-time (head `7adab031b`)

Cards 1 to 5.
`Identity` carries the commit time and gains `Clean` and `Label`.
`buildinfo` gains a production channel stamped by the deploy tool.
`stencilstore` gains the three modes, writer keys in the banner, the forward-only refresh rule and `WritesDue`.
`StencilSource` orders by ancestry or commit time and stamps the running binary.
`gitrepo` gains the upstream SHA, commit detail, cherry-pick and the rebase-failed sentinel.
Deviation: `internal/cliwire/doc.go` and `internal/cliwire/seam_enforcement_test.go`, because `buildvcs` joined `cliwire`'s import allowlist.

## Batch 06 — seed-commit-signature (head `ddee050b6`)

Cards 6 to 9.
The shared binary label, seed and sync subjects and the seed-commit recognizer landed.
`Bolt.PullThenCommitWritten` and the gated seed-commit drop landed.
The root pre-run gates on a due write, pulls Bolt first and labels its commit.
`lyx stencil sync` commits under its own labelled subject.
Deviations: `internal/fabricengine/mutation.go`, `stencilcommit.go`, `bolt.go` and `destroy.go`.

## Batch 10 — hubreconcile-pull-first (head `a9bf0d982`)

Cards 10 to 13.
The hub-wide write pulls first and leaves the stamp absent when it cannot land.
Hub-config and warp-binding commit or push failures fail `lyx fabric reconcile`.
Store gc skips under another pair's live session and prunes day-old loose objects.
Bolt's push drops seed commits and retries once on a rebase conflict, recorded in each fabric verb's envelope.
Deviation: `internal/fabriccli/adoptremote_integration_test.go`.


## Integration suite triage

The plan-level `## verify:` suite failed, but webster's triage did not attribute the failure to this run.

Flaky (passed on rerun):

- `github.com/Knatte18/loomyard/internal/loomcli.TestLoomResumeReconcilesTheHubConfigBeforeArming`
