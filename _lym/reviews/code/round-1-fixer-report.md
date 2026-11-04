# Fixer report — code review round 1

All eight findings are fixed.
`lym gate test` over `landingshed`, `pawlengine`, `hubgeom`, `loomcli` and `beamcli` at tiers 0 and 1 passed after the first round of edits, except one test row of mine whose expected stop count was wrong; I corrected it.
After the self-check's changes, `pawlengine` and `beamcli` at tiers 0 and 1 passed again.

## Changes

- **F1** (fixed as proposed): `answerReopen` in `internal/landingshed/prreview.go` has a merged case that sets the fate to "pull request #N is already merged" and makes no GitHub write, and its record-removal condition now includes a merged pull request.
  `TestPRReview_Reopen` has a row with a merged `Status` that asserts only `Status 7`, no record left and a Stuck reroute naming the merged pull request.
  The `answerReopen` doc comment and `internal/landingshed/doc.go` say "closed or merged".
- **F2** (fixed as proposed): `TestRemoveReady` in `internal/pawlengine/ready_test.go` covers a record naming the slug, an absent record and a record naming another slug.
  `TestHubLandDeps_UnreadyCommitsTheRemovalOfTheReadyRecordAndSucceedsAgain` in `internal/beamcli/dobbydeps_integration_test.go` runs the production `Unready` over a real pair: the removal is committed and a second call with no record left succeeds.
- **F3** (fixed as proposed): `TestReadReopenRequest` in `internal/loomcli/wiring_beam_test.go` has rows for no request file, a request of another run, no seed, and a matching request with and without `Close`.
- **F4** (fixed as proposed): `TestPRReview_Reopen` makes the reviewed record's strand live and counts stops through the sessions fake (`wantStops`); the draft, close, closed-on-GitHub, merged and transient-failure rows assert exactly one stop.
  The fake gained `stopErr`, and a row where the stop fails expects an error, no GitHub call and the record unchanged; its `wantStops` is 2, because a `Call` that returns an error ends its session, which stops the still-live strand once more (the field comment says so).
- **F5** (fixed with my own change, after the self-check; see below): `enter` calls `answerReopen`, which reports whether the entry may go on, and `entered` is set only after it returns true.
  A reopen request naming this run whose removal fails starts no dobby, posts one notice naming the file, and is retried at the next wake.
  A request of another run whose removal fails is logged and the entry goes on.
  Two `TestBeamAwait` rows pin both cases over a read-only directory (skipped for root and on Windows).
  `internal/pawlengine/doc.go` and the doc comments say so.
- **F6** (fixed as proposed): the `Unready` closure in `internal/beamcli/deps.go` asks `AnchoredPathHasContent` before `CommitAnchoredPaths` and returns `removed, nil` without a commit when the landing directory holds nothing.
  `internal/beamcli/doc.go` says so, and the tier 1 test of F2 covers the repeat call.
  I checked in a scratch repository that `git commit -- <empty untracked directory>` fails with "pathspec did not match", so the guard is needed.
- **F7** (fixed as proposed): `TestDobbyGeometry` checks `Paths.ReopenRequest` too, and its comment no longer counts the files.
- **F8** (fixed as proposed): the `beamDescribeDeps` doc comment says an empty `parentBranch` is refused when the beam's commits are read.

F5, F6 and F2 have one commit each for the first round, and a later commit for the self-check changes.
F1 and F4 share one commit, and F2 and F6 share another, because each pair edits the same test file and cannot be split without an interactive patch.

## Self-check

Lens reports are saved beside this file as `round-1-fixer-report.lens-<lens>.md`; every fork returned a report.

- fix-proposal, MEDIUM: F5 as proposed holds the whole row, land requests included, when a removal keeps failing, and also for a stale request of another run, with only a Warn to show for it. Fixed with my own change (above): only a request naming this run holds the entry, and it posts one notice naming the file.
- fix-code-correctness, LOW: the same stall as fix-proposal's finding. Resolved by the same change.
- fix-code-correctness, LOW: the `Unready` closure returned `(false, err)` when the content probe failed after a removal, dropping `removed`. Fixed: it returns `removed, err`.
- fix-code-correctness, LOW: the tier 1 test might pass without F6's guard if git tolerated an empty pathspec. Not changed: the scratch-repository check above shows git refuses it, so the second `Unready` call fails without the guard.
- fix-proposal, LOW: the `wantStops: 2` row ties the test to the session end's second stop. Not changed beyond the field comment that names the reason; the draft and close rows pin that the reopen stops the strand exactly once, so the stop-failure row keeps its job of pinning the error, the missing GitHub call and the unchanged record.
- fix-consistency: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
