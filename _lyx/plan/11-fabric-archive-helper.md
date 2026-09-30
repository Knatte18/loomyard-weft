# Card 11 — fabric-archive-helper

**Create:**
- `plan:internal/fabricengine#archiveWeftTip` -> `func archiveWeftTip(rec *Mutations, l *lyxcwd.Location, slug, weftBranch string) (tag string, skippedReason string, err error)`
- `plan:internal/fabricengine#KindTagPushed` -> `const KindTagPushed Kind = "tag_pushed"`
- `internal/fabricengine/archive.go`
- `internal/fabricengine/archive_integration_test.go`

**Edit:**
- `internal/fabricengine/mutation.go`

**Uses:**
- `internal/fabricengine#Kind`

**Intent:**
Add the one helper every weft-branch teardown of an existing pair calls first: tag the weft tip under an archive namespace and push the tag to the weft origin, so the run records on that branch stay reachable after the branch is deleted, without blocking a later create of the same slug.

`archiveWeftTip` in the new `archive.go` (tag creation and push are not destructive, so it lives outside `destroy.go`):
- Resolves the weft repo root and its `origin` remote.
  No origin configured: returns no tag, a `skippedReason` worded like `RemoteSkippedReason` ("no archive tag pushed: the weft repo has no origin remote configured: …"), and no error.
- Resolves the tip: the local `refs/heads/<weftBranch>` when present, else the origin's copy fetched into the weft repo; neither present returns no tag, no reason and no error (nothing to archive).
- Names the tag deterministically `archive/<slug>/<first 12 hex of the tip SHA>`, creates it on the tip when absent, accepts it when it already points at the tip (a resumed teardown), and errors when an existing tag of that name points elsewhere.
- Pushes `refs/tags/<tag>` to origin; a tag already on origin at the same commit is success, so the call is idempotent.
  Any push failure, including an origin that is configured but unreachable, is a returned error naming the tag and the remote.
- Records `KindTagPushed` on `rec` only after the push observably succeeded (Mutation Record Invariant), with the tag as detail.
All git runs through `gitexec.Run` in this package (Fabric Git Invariant).

Declare `KindTagPushed` in `mutation.go` among the hand-recorded kinds and update the header comment's split of auto-recorded versus hand-recorded kinds.

Tests in `archive_integration_test.go`, over a hubforge hub whose weft repo has a bare origin (Hermetic Git Test Environment Invariant already applies to the package): the tag exists on origin at the tip after the call and is returned; a second call on the same tip succeeds and records no duplicate push failure; a weft repo with no origin returns a skip reason and creates no tag; an unreachable origin URL returns an error and leaves no tag on any remote; a branch present only on origin is archived from the fetched tip; and a clashing tag of the same name at another commit is an error.

**ImpactSummary:** Adds one mutation kind and one unexported helper with no callers yet.
