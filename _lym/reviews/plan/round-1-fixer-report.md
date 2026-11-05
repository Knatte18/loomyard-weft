# Round 1 fixer-report — plan

Every finding's premise checked out against the code it cites.
`lym loom validate-plan` answers `ok: true` after all changes.

## Changes

- **F1 (BLOCKING), fixed differently.**
  Card 14 creates `parentRuleRequired(ctx, origin, parentBranch, open)`, and `PullRequestMode` in `hubLandDeps` calls it with the beam's origin facts and a closure over `pullrequest.Open`.
  It reads GitHub's rules only when `landingshed.OriginMissing(origin)` is empty.
  A missing origin fact, an unusable token, or a failure that is not transient reads as no rule, logging one `logger.Warn` where the read failed, as `resolveWithParentRule` reads it (`pullrequestmode.go:113-139`); a transient failure is returned.
  Card 14's Uses gains `plan:internal/landingshed#OriginMissing`, `internal/pullrequest#API` and `internal/pullrequest#Open`.
  Card 15's refusal 5 refuses a `PullRequestMode` error, naming the cause and `lym beam ready <slug>` again, and `TestMarkReady` gains that refused row.
  Not made as proposed: the `TestMarkReady` passing rows (an off parent with a non-GitHub origin, an on parent with a token failure) and the `TestPRClose` row for a non-GitHub off parent.
  Both tests run over `newLandWorld`'s fakes, which stand in for the `PullRequestMode` closure, so those rows would pass whatever `hubLandDeps` does.
  The gating is pinned where it lives instead:
  - `TestParentRuleRequired` (card 14, tier 0) has rows for an origin not on GitHub, no upstream, a token failure, a non-transient failure, a transient failure, and rules read true and false.
  - `TestHubLandDeps_ReadyReads` (card 15, tier 1) drives the real `PullRequestMode` over a hub whose origin is a local path: an `off` parent answers with no rule and no error.
- **F2 (MEDIUM), fixed as proposed.**
  Card 1's Intent names the arm-time rename (`loomrecipe.RetiredRows` through `loomCLI.renameRetiredRows` and `shedengine.RenameProducers`) and states how it differs from the recipe's `retired` map.
  Card 5's pawlengine bullet says `RetiredRowNames`' doc reads the `retired` map and never `RetiredRows`.
  A new card 5 bullet has `internal/loomrecipe/doc.go` and `internal/loomcli/doc.go` keep the two mechanisms apart.
- **F3 (MEDIUM), fixed as proposed.**
  - Card 5 adds `internal/battencli#runTakesNoSlot` under Edit, and its Intent rewrites that doc and `loomrecipe/doc.go`'s pawl paragraph: six rows, no reroute and no `lym beam reopen`, hold from Finalize.
  - Card 7's Intent rewrites `dobbyengine/doc.go`'s slot rule to say Finalize.
  - Card 12 adds `internal/loomcli/pawlwiring.go` under Edit and drops "beyond the BeamDescribe row" from its header.
  - Card 15 adds `internal/beamcli#mergeParentCmd` under Edit and rewrites its Long and `beamcli/doc.go`'s merge-parent paragraph.
- **F4 (MEDIUM), fixed as proposed, with one part done differently.**
  Card 15's refusal 5 and card 14's `prClose` pass `deps.BeamHasCommits(slug)` as `taskHasCommits`.
  Card 15 refuses a `BeamHasCommits` error naming the cause and the rerun, and gains the passing row for an empty beam under a required parent, plus a `BeamHasCommits` error row.
  Done differently: in `prClose`, a `BeamHasCommits` error still closes and drops, with the close-only comment, and reports the error, as a failed mode read already does there.
  Refusing instead would make two failed reads of the same verb behave differently.
  Card 14's comment rule now keys on the decision ("a no", including an empty beam) rather than on the mode, and `TestPRClose` gains the empty-beam and `BeamHasCommits`-error rows.
- **F5 (MEDIUM), fixed as proposed.**
  Card 12's `judgeAwaitingRun` lets a run at Beam-Await pass in every state but done, and says why: Beam-Await halts through an error stop that is recorded as failed.
  `TestDescribe`'s halted passing row is now failed.
- **F6 (LOW), fixed as proposed.**
  Card 10 adds `internal/pawlengine#awaitRun.accept` under Edit.
  Its Intent states the removal order (await marker, then dobby-stop marker, then land request) and the two states a crash can leave.
  `TestBeamAwait` gains the row with the await marker gone, the land request standing and the ready record standing, which is accepted and Done.
- **F7 (LOW), fixed as proposed.**
  Card 5's `TestBeamAwait` sentence and card 10's row now name Parent-Align, the retired hold an old binary's accept wrote, in place of PR-Review.
- **F8 (LOW), fixed as proposed.**
  Card 3's `RequiredChecks` also reads every page of `Repositories.GetCombinedStatus` at head and falls back to a context's commit status, reading `pending` as empty.
  A context is `missing` only when neither a check run nor a commit status carries it.
  `TestClient_Calls` gains a row whose required context appears only as a commit status.

## Self-check

- fix-consistency, MEDIUM, `recheckStanding` kept the old removal order: fixed as proposed. Card 10's `recheckStanding` now removes the await marker before the dobby-stop marker, and the accept bullet says why.
- fix-consistency, LOW, card 3's `Check` parenthetical still said `missing` when no run exists: fixed as proposed.
- fix-consistency and fix-claims, LOW, `pullrequest.Open` returns `*Client`, not the `open` func type (duplicate across the two lenses): fixed as proposed. Card 14 now passes a closure over `pullrequest.Open`.
- fix-consistency, LOW, card 15 names `parentRuleRequired` without listing it under Uses: fixed as proposed.
- fix-affected-set, MEDIUM, `beamForkPrecondition`'s doc (`battencli/beamfork.go:48`) still says "from Parent-Align on": fixed as proposed in card 7, which already edits that member.
- fix-affected-set, LOW, `shedengine/doc.go`'s `RenameProducers` paragraph doesn't tell the two retired-row mechanisms apart: fixed as proposed in card 1, which already edits that doc.
- fix-proposal, MEDIUM, `BeamHasCommits` never says whether records-only commits count: fixed differently.
  The lens proposed counting only non-records-only commits.
  Finalize's `TaskHasCommits` counts every commit the local parent lacks (`internal/landingshed/deps.go:361`, `internal/loomcli/landingdeps.go:361`), and a pull request opens on any non-empty branch.
  So card 13 now defines `BeamHasCommits` as that same read, with records-only commits counted, and `pr open`, `pr close`, `ready` and Finalize judge an empty beam alike.
- fix-proposal, LOW, `GetCombinedStatus` read without "every page": fixed as proposed in card 3.

## Disputed

None.

## Deferred

None.
