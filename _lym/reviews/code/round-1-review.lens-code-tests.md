```yaml
lens: code-tests
outcome: reported
```

Task: list, per behaviour the diff adds or changes, the test that pins it (or none), for the code-tests lens.

## Per behaviour: the test that pins it, or none

- **Finalize commit title ends ` (#N)`**: pinned by `TestFinalize_PullRequest` (`wantGitHubMerge`, `finalize_pr_test.go:34-35`).
- **Beam-Describe `## Changes` list** (pull request entry, parent-merge and direct entries, oldest first, first-sentence summary, ellipsis cut, heading-paragraph skip, 12-character object name): pinned by `TestBeamDescribe` rows `PullRequestsAndDirectCommitsAreListedOldestFirst`, `ALongSummaryIsCutWithAnEllipsis` and `ProjectDecisionsFollowTheChanges`. A title that carries `(#N)` inside it and not as a suffix has no row. This is marginal and not filed.
- **`beamCommits` first-parent line** (members and parent merges listed, merged-in commits left out, oldest first): pinned by `TestBeamCommits_ListsTheBeamsOwnLine` (tier 1). The `len(added) == 0` early return has no row. That is also marginal and not filed.
- **`Commits` seam rename**: pinned by the seam lists in `wiring_beam_test.go` and the `intentionallyNil` tables.
- **`lym beam reopen` refusals, accepted rows, `--close`, halted run, failed ready removal**: pinned by `TestReopen` (`land_test.go`). Failures of `Machine`, `ReadOwners`, `ChildStatus`, `RunID` and `WriteReopen` have no row. That is the same wrapping pattern as `land` and is not filed.
- **Production `Unready` and `WriteReopen` wiring** (`beamcli/deps.go:133-153`, which commits even when no record was removed): **none**, see F1.
- **`pawlengine.RemoveReady`** (removes only a record naming the slug, leaves an absent or other-slug record): **none**, see F1.
- **`reopenCmd` and its `--close` flag wiring** (`cli.go`): none found in `beamcli` tests (grep). Low value, filed with F1.
- **Reopen request `Write`/`Read`/`Remove`**: exercised through `beamawait_test.go` via `world.reopen`.
- **`Paths.ReopenRequest` required seam** (`deps.go` `pathsSeams`): `world_test.go` and `fixture_test.go` fill it. I found no row that drops it and expects a wiring error. That matches the existing treatment of the other path seams and is not filed.
- **`PawlPaths` sets `ReopenRequest` under `.lym/beam/`**: **none**, see F2.
- **Beam-Await answers a reopen request**: pinned by four `TestBeamAwait` rows (dobby down and started, live dobby, failed start naming `lym beam start`, another run's request removed alone). A failed `DobbyLive` read, which assumes live, has no row. That is minor and not filed.
- **Recipe edge `PR-Review on_reroute: Beam-Await`**: pinned by `pawlrecipe_test.go` and `wiring_beam_test.go`.
- **`readReopenRequest` in `loomcli/landingdeps.go`** (no file reads as none; a request naming another run reads as none; a missing seed reads as none; a matching request is passed through with `Close`): **none**, see F3.
- **PR-Review `answerReopen`, GitHub effects and record handling**: pinned by `TestPRReview_Reopen` rows (draft with comment and record reset, already a draft, `--close`, closed on GitHub, no pull request, request seen at a wake, transient failure on `Status`).
- **PR-Review `answerReopen` stops the live pr-reviewer session**: **none**, see F4. The `reviewed` record carries a `StrandGUID`, but the test asserts only GitHub calls and the record. `SessionLive` and `StopStrand` appear in `prreview_test.go` only in the wiring-nil table (line 450) and the fixture wiring (line 723).
- **`answerReopen` on a pull request GitHub reports merged** (kept, record reset, reason "is merged"): none, see F5.
- **`answerReopen` with a failing `SetDraft`, `Comment` or `Close`** (`githubFailure` judgment): only a `Status` failure is tested. These share the existing `githubFailure` path and are not filed.
- **Re-entry marks a draft ready before editing**: pinned by `TestPRReview_Call` row `ReentryOnADraftMarksItReadyForReviewBeforeTheEdit`.
- **`pullrequest` client `Comment`, `SetDraft`, `Status.Draft`, GraphQL error handling**: pinned by `TestClient_Calls`. A non-2xx GraphQL transport failure has no row, but `SetDraft(99)` covers the failing pull request read.
- **Doc comments, `pawl-recipe.yaml` comment, `docs/overview.md`**: prose, exempt.

## Findings

**F1 — MEDIUM, design.** `beamcli/deps.go:133-153`, `pawlengine/ready.go:1071-1087`. `RemoveReady` and the production `Unready` have no test. Unready is the step that keeps the dobby from landing the beam again, and it commits even when no record was removed. `TestReopen` fakes `Unready` entirely. Rows are needed for removing a record that names the slug, leaving an absent record, leaving a record naming another slug, and returning `removed` accordingly. Add them to the existing ready-record test, or a small `RemoveReady` table, at tier 0 with `t.TempDir`. The commit-regardless behaviour needs a tier-1 case over a real pair, or an assertion that the removal is committed.

**F2 — LOW, design.** `hubgeom/dobbygeom_test.go:108-115`. `TestDobbyGeometry` iterates only `LandRequest`, `DobbyStopMarker` and `AwaitMarker`, and its comment still says "three files". Add `"ReopenRequest": paths.ReopenRequest` to that map, which is a row in an existing test and not a new test, and update the comment.

**F3 — MEDIUM, design.** `loomcli/landingdeps.go:841-854`. `readReopenRequest` has no test. It is the only place that decides a request belongs to the run (`seed.StartedAt != req.RunID`) and that no request reads as none. A bug here either makes PR-Review reroute on another run's request or never reroute. Add a table over a real or temp-dir pair covering no file, a matching run id with `Close`, another run id, and a missing seed. This can sit in `wiring_beam_test.go` or a tier-0 test using `shedrun` fixtures.

**F4 — MEDIUM, design.** `landingshed/prreview_test.go:548-659`, `TestPRReview_Reopen`. The decision record requires "Stops the pr-reviewer", and the production code calls `stopLiveSession`. No row makes `SessionLive` return true and asserts that `StopStrand` ran. Break the stop and every row still passes. Add an assertion in the draft row, or a new row, with the fixture's `sessions` fake reporting a live strand and checking that the stop was recorded. Also check that a failed stop returns an error and leaves the record.

**F5 — LOW, design.** `landingshed/prreview.go:677-709`. `answerReopen` with a pull request GitHub reports merged (neither open nor closed) falls through `switch`. The record is kept and reset, the reroute reason is "is merged", and no comment or draft call is made. No `TestPRReview_Reopen` row pins this, so the intended behaviour is unpinned. Add a row with `Status` returning `StateMerged`, asserting only the `Status 7` call, the reset record and the reroute reason.

Tool calls (1 of 6):
1. `grep`/`sed` over test files for `RemoveReady`, `readReopenRequest`, `ReopenRequest`, `StopStrand`/`SessionLive`, the `dobbygeom_test.go` span, and `beamcli` reopen tests. My context had no tests for these and I needed to confirm they are absent.
