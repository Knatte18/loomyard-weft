# Card 8 — darn-landing-tree-scope

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#Publish.Call`
- `internal/landingshed/doc.go`
- `internal/loomcli#verifySource`
- `internal/loomcli#darnVerifySource`
- `internal/loomcli#landingDeps`
- `internal/testkit/envkit#nilLegal`
- `internal/landingshed#TestPublishVerify_Proceeds`
- `internal/landingshed#TestPublishVerify_HardErrorsAreReturned`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `plan:internal/loomcli#TestDarnTreeScope_ClassifiesAtEachCall`

**Uses:**
- `plan:internal/loomcli#darnTreeScope`
- `internal/darnengine#VerifyCommand`
- `internal/landingshed#newPublishVerifyFixture`
- `internal/landingshed#verifyGate.checkPublishVerify`
- `internal/verifytree#WritePublishFailure`
- `internal/verifytree#ReadPublishFailure`

**Intent:** Carry D2's classification to a darn run's landing sites (D4).

`verifySource` gains `publishSkip func() (bool, error)`, nil for loom's source.
`darnVerifySource`'s `command` resolves at each call: the tree-scoped command when `darnTreeScope` answers tree-scoped, else `darnengine.VerifyCommand`, so Merge-In, which calls it after its merge, verifies the task's own side against the parent tip just merged, and Finalize resolves the same way and skips through the verified-tree record on the tree Merge-In verified.
Because the writer gate records its pass under the same deterministic string, a Merge-In with no Publish failure record on the tree a writer gate passed skips through the record.
Its `publishSkip` answers whether `darnTreeScope` classifies the diff as tree-scoped at that call.
`landingDeps` passes it as `Deps.PublishVerifySkip`, a new `func() (bool, error)` field documented as nil for loom.
`Publish.Call`, after `holdParent`, calls the seam when it is non-nil and `publish_verify` is set: true takes the path of an unset `publish_verify`, so no `publish_verify` runs and the Publish failure record is removed before the push; a seam error is Publish's returned error, read the way the merge-commit read fails, with nothing verified or pushed; false runs `publish_verify` as today.
The seam is classified at each Publish, against the merge base Merge-In's merge left, so a run tree-scoped at one Publish whose PR-Gate rework adds a production path runs `publish_verify` at the next.
What the skip lets through is bounded by D4: a tree-scoped darn run lands with no `tmux`-tier run and with only the tree-scoped command at Merge-In, the hub's whole statement of what such a diff needs, and a nil seam, loom's, runs `publish_verify` exactly as before.
`doc.go`'s `publish_verify` paragraphs say so.
`nilLegal` gains `Landing.PublishVerifySkip`.

`darnVerifySource`'s doc comment says its command resolves per call to the tree-scoped command or `darn.yaml`'s `verify`, and names its `publishSkip`.

`TestPublishVerify_Proceeds` gains rows:
- a seam answering true with `publish_verify` set and a Publish failure record written before the call calls no verify, pushes, and leaves no record;
- a seam answering false with `publish_verify` set runs it once at site `Publish` with the plan's command as base command, and pushes;
- one fixture called twice, its seam answering true at the first `Call` and false at the second, runs `publish_verify` only at the second, as after a PR-Gate rework that adds a production path;
- a seam that fails the test when called, with `publish_verify` unset, runs no verify and pushes.
`TestPublishVerify_HardErrorsAreReturned` gains a row: a seam error is returned naming its text, with no verify and no push.
`TestLandingDeps_EveryFieldPopulated` skips `PublishVerifySkip` in its walk, since the loom source it is told leaves it nil, and asserts it is nil there.
`TestDarnTreeScope_ClassifiesAtEachCall` also builds `landingDeps` over `darnVerifySource` for the pair and asserts at each of Card 7's steps that `VerifyCommand()` returns the gate seam's scoped string and `PublishVerifySkip()` true where the seam answers tree-scoped, and that `VerifyCommand()` returns the hub's `verify` and `PublishVerifySkip()` false where it answers not tree-scoped, the merged production resolution, the rename and the emptied `tree_scope_verify` included.

**ImpactSummary:** Publish gains one optional seam that only a darn run wires; Merge-In and Finalize keep their code and read a source that now resolves per call, and every loom landing behaves as before.
