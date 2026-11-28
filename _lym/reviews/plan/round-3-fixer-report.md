---
dispositions:
  - id: F19
    disposition: fixed
  - id: F20
    disposition: fixed
  - id: F21
    disposition: fixed
---

# Plan fixer-report, round 3

## Changes

- **F19** (card 12):
  - Removed the Create lines of `newReplayHub`, `pushPrimeRecords`, `commitOnOrigin`, `endEntryOnOrigin` and `addPushedPair`.
  - Added the Create line `plan:internal/gitkit#CommitOnBare`, the Extends line `internal/gitkit/doc.go`, and the Edit lines `recordsFixture.originCommit` and `originCommitFile`, which call the new export.
  - Replaced the restating sentence with the rule that shared git plumbing lives in `internal/gitkit`, and named the `replay_integration_test.go` edit as a departure from D9's own-files criterion.
  - Replaced the `newReplayHub` bullet: the rows call `boardkit.InstallReplay` and `hubforge.CopyHub`, then `hubforge.AddPair` and `lymcwd.ResolveWorktree`, then one `gitkit.Git` `push --set-upstream`, then `gitkit.CommitOnBare` over `boardkit.BoardFile`.
  - The unverified heddle fact holds: no file holding a moved test mentions heddle.
    The card says so, and says that a row changing `heddle.json` would register heddle's replayer in its own body.
- **F20** (card 12):
  - Removed the four `pushkit` Create lines.
  - Removed `requireOneFailedPush`, `requireNoFailedPush`, `failedPushesOf`, `countingBackoff` and `requireEntry` from Delete, and the seven retarget-only lines from Edit.
  - Replaced the `pushkit` bullet and its D9 departure with: fabric's helpers stay, and moved rows assert inline over `ListFailedPushes` and the report's `Entries`, with an inline backoff counter.
  - Dropped `pushkit` from the `teardownRecorder` bullet and replaced the kit in the ImpactSummary with gitkit's new helper.
- **F21** (cards 12 and 9):
  - `teardownRecorder` is now a recording wrapper over the real seam from `pairteardown.New(h.Location)` and `OriginSeams(pairteardown.HubSeamOptions(h.Location))`, with `LivePair` set to nil.
  - `TestReconcileOrigin_EndedRuns` asserts the mutations the real teardown records and no scripted tags, kept lines or entries.
  - `TestReconcileOrigin_OwnedElsewhere` takes `OwnerOf` from `pairteardown.OwnerReader` with a fixed identity, and its error row uses board bytes that do not decode.
  - The empty-subject row stays with card 9's `TestClassifyEntries`, and card 9's description now names it.
  - The intent paragraph names `internal/pairteardown` among `boardengine_test`'s imports.
  - The card keeps the review's two unverified facts as Unverified: whether the real teardown runs in tier1 with no tmux server, and which board fields the owner reader needs.
- **Self-check fixes** (card 12):
  - Added Uses lines for `gitkit#Git`, `lymcwd#ResolveWorktree`, `pairteardown#New`, `#HubSeamOptions`, `#OwnerReader` and `#Teardown.OriginSeams`.
  - Added the gitkit doc addition's sentence.
  - Replaced "leave `LivePair` nil" with "set `LivePair` to nil on the returned seams".
  - The owner reader now takes a `parentOf` naming each row's parent, so the beam and root subject rows come from the real reader's adopt-subject read.
  - The no-origin-head row observes "never asked" through a wrapper that records each call and passes it to the real reader.

## Self-check

- fix-consistency, LOW, Uses list misses the exports the new text calls: fixed.
- fix-consistency, LOW, the gitkit doc Extends line has no matching sentence: fixed.
- fix-consistency, note, card 10 line 65 disagrees with card 12 on the run-scope and earlier-snapshot-tag rows: out of scope. The disagreement predates this round, and this change did not touch or reach that text.
- fix-claims: no findings.
- fix-proposal, LOW, "leave `LivePair` nil" misreads `OriginSeams`, which always fills it: fixed.
- fix-affected-set, MEDIUM, the beam and root subject rows have no source under the real reader: fixed.
- fix-affected-set, LOW, the never-asked row has no way to observe the real reader's calls: fixed.

## Disputed

None.

## Deferred

None.

## Rulings

None.
