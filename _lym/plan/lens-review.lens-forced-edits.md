```yaml
lens: forced-edits
outcome: reported
```

Forced-edits lens findings:

- **BLOCKING** (Card 10, D5's way forward): the new `MidMergeRecordsUnsettled` kind falls into the foreign-merge `default` branch in two callers no card lists. Those branches tell the operator to "conclude or abort it with git", while D5 names only `lym fabric merge --abort` for a merge stopped before step 3. They also lose the own-merge-in-leftover handling the `MidMergeParked` case gives a merge-in of the parent.
  - Evidence: `internal/landingshed/premerge.go:47-55` (`preMerge.clear` switch); `internal/loomcli/start.go:233-249` (`loomCLI.refuseOverUnfinishedMerge` switch).
  - `fabricengine.MidMerge` is wired into loomcli at `internal/loomcli/landingdeps.go:205`.
  - Card 10 lists only `Finalize.dirtyParentReason` and `Finalize.parkedMergeWayForward`.
- **MEDIUM** (Card 10, new refusal reason): `MergeContinue`'s new refusal, which names the unfinished records step, is a guard reason. A guard reason is a pinned const-block member.
  - No card lists `internal/fabricengine#mergeReasonAlreadyInProgress`'s const block in `mergeerrors.go:22-33`, nor `internal/fabricengine#TestMergeErrors_PinnedStrings` or `internal/fabricengine#TestMergeVocabulary_GuardReasonSetMatchesConstBlock`.
  - The vocabulary tests pin the reason set, so either the new reason is added to the set and its tests, or the card states it is no `MergeGuardError` reason.
  - Card 9's new `*ErrMarkerChange` likewise faces `internal/fabricengine#TestMergeVocabulary_ErrorsAreSideFree`, which no card lists. Evidence: `mergevocab_test.go:124-152, 229`.
- **MEDIUM** (Card 14, act-rule way forward): `FailedPushesOwedBy` reading `operator_unpushed` as owed sets `PushOwed` through `internal/pairteardown#hubRunReader`. The act rule's refusal names `lym fabric push` in the owed worktree (`pattern/PATTERN-run-fsm.md:34`). For an `operator_unpushed` hold, D7's way forward is the operator's `git push` in the prime. No card touches `internal/runfsm` or `internal/pairteardown/runread.go:144`, so the refusal names the wrong step.
- **LOW** (Card 11, described behavior): two descriptions keep the old skip after the seam waits in its place.
  - `docs/overview.md:339` still says statuscommit will "skip while mid-merge". No card lists it.
  - `internal/loomcli#commitStatusDeps`'s doc (`internal/loomcli/wiring.go:64-69`) says "a mid-merge state skips the step". Card 11 edits that type, but its Intent does not say the doc changes.
- **LOW** (Card 5, hubforge one-repository hub): `internal/hubforge#SeedConfig` runs `git add .` at `h.PrimeRecords()`. In a one-repository hub that path is the prime code worktree (Card 3), so a seed would sweep code into the commit. No card adjusts it or bars it for `NewOneRepoHub`. The same holds for `internal/hubforge#ReanchorPair`. Evidence: the hubforge lookup, `seed.go:14-50`.
- **LOW** (Card 10, constant form): `MidMergeRecordsUnsettled` is declared as `= 3` outside the `iota` block that starts at `internal/fabricengine/midmerge.go:17`. Extending that block keeps it in step if a kind is added.

Tool calls:
1. `lym quarry resolve` of the listed glyphs: my context had no quarry answers for the lesser-known Edit glyphs.
2. Grep of `MergeActive:` users, `MidMerge` kind switches, the `protectedBranch*` references and the mid-merge-skip mentions: to find callers outside the cards.
3. Read of the `premerge.go` and `start.go` switch bodies, plus doc mentions: to see where the new kind falls through.
4. Grep of the `runCloneWithReset` and `cloneHub` callers, the `MidMergeKind` iota and the destroy test references: to check re-sign coverage.
5. Grep of the destroy and live-state test files, plus the `MidMerge` and `MergeStateActive` callers outside fabric: to check registration tables.
6. Read of the `livestate_verbs_test.go` header: to judge whether it is an executor registry. It is not.
