# Lens review

Every lens returned a report; each is saved beside this file as `lens-review.lens-<lens>.md`.
Findings two lenses raised alike are merged and listed once, under the first lens.

## coverage

- coverage 1, MEDIUM, D2 adopt criterion untested — fixed as proposed: card 7 adds a tier1 row to `TestAdoptIntegration_SecondCloneTakesOver`, where the second clone adopts after an unpushed local transition and resumes at origin's last transition.
- coverage 2, MEDIUM, batten versus loom pusher target untested — fixed differently: the proposal to assert the target in each deps builder needs a spawn seam per builder; instead both builders make the same call, `runpusher.TargetOf(location, resolved run id)`, and `TargetOf` gives the prime an empty slug (card 5), so no builder chooses a target and `TestStart`'s target rows pin what each gets.
- coverage 3, MEDIUM, D8 `PushOwed` unset untested — fixed differently: no `ReadRun` row, since `ReadRun`'s owed read is exactly `FailedPushesOwedBy` (`internal/pairteardown/runread.go`, `hubRunReader.owedPush`); card 1's rows now assert `FailedPushesOwedBy(hub, slug)` returns nothing after each owed push in the mode.
- coverage 4, MEDIUM, `recordOwedRecordsPush` skip unreached — fixed differently: card 1 reruns the existing `BackoffSpent` row of `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide` in the mode, which reaches `recordOwedRecordsPush` through the records loop's spent backoff, rather than a new fixture.
- coverage 5, MEDIUM, failed archive and failed hand-over untested — fixed as proposed: card 3's `TestHandOverTips` and card 4's `TestPushRun_HandsOverOnATakeover` gain rows with a pre-receive hook refusing tags.
- coverage 6, MEDIUM (also row-validity 7), `TestPusherDeps` checks presence only — fixed as proposed: `pusherDeps` takes a `pushRun` seam, and the test asserts the slug and `RunPush` (`OwedUnrecorded`, `InsidePair`, waits) per target.
- coverage 7, MEDIUM (also row-validity 8), dobby restart untested and "not owned elsewhere" beyond the record — fixed differently: the restart goes through a new `battencli.RestartPusher` with `TestRestartPusher` covering its start, failure episode and recovery, and card 8 drops the owned-elsewhere condition so every pair standing here is restarted, as D5 names; the prime tick calls the same helper, so it joins no error.
- coverage 8, MEDIUM, pre-run skip of `pusher` untested — fixed as proposed: `TestResolvePersistentPreRun_SkipsSlugGroups` gains a `pusher --worktree <missing>` row, which reaches the resolve error only past the skip.
- coverage 9, LOW, `Run` read and lock errors untested — fixed as proposed: `TestRun` gains those rows.
- coverage 10, LOW, no-`Warp-SHA` row missing — fixed as proposed in `TestRunCommitsAhead`.
- coverage 11, LOW, `withoutFinalizeEntered` keep untested — fixed as proposed: a prime row whose handed-over run stands at its Land row.
- coverage 12, LOW, teardown wire's gone-pair branches untested — fixed as proposed: `TestBattenIntegration_Rows` gains a resumed Worktree-Teardown row with the pair gone.
- coverage 13, LOW, loom resume order unasserted — fixed as proposed: the `TestResumeVerb` row records the start before the wake.

## row-validity

- row-validity 1, BLOCKING, nil teardown seams panic — fixed as proposed: a nil `Push` pushes nothing and a nil `Level` reads level.
- row-validity 2, MEDIUM, no lock fake records release order — fixed as proposed with a new `recordingPrimeLock` appending lock and release to the recorder.
- row-validity 3, MEDIUM, new teardown test should be rows — fixed as proposed: `TestWorktreeTeardown_PushesWithThePrimeLockReleased` is dropped and its rows join `TestWorktreeTeardown_TearsDownInOrderOnceTheLockIsFree`, `TestWorktreeTeardown_ShutdownFailureNeverCallsRemove` and `TestWorktreeTeardown_PrimeLockDispositions`.
- row-validity 4, MEDIUM, no prime fixture — fixed as proposed: card 3 creates `newPrimeRunsAhead`.
- row-validity 5, MEDIUM, takeover-read-failure input unnamed — fixed as proposed: the row names invalid JSON in the run's status on origin's tip, which `originStatusAtTip` returns as a parse error (`internal/fabricengine/originreads.go:79`), so the premise that it may read tolerantly is answered there.
- row-validity 6, MEDIUM, `IsPushOwed` tested from runpusher — fixed as proposed: card 3 adds rows to `TestIsPushOwed`, and card 5 drops its `TestRun` row.
- row-validity 9, MEDIUM, nil `startPusher` runs real reads — fixed as proposed: a nil seam starts nothing, and `battenCLI.wire` and `loomCLI.wire` fill it.
- row-validity 10, LOW, runproc row repeats — fixed as proposed: card 5 drops its `TestEndSubject_EndsTheRunsThenTheSubjectAndItsDobbyLast` edit and cites the test under `Uses`, since its `RolePush` record under `HubOwner` (`internal/runproc/end_test.go:480`) already outlives every row.
- row-validity 11, LOW, transport URL — fixed as proposed: the row uses `refusedOriginURL`.
- row-validity 12, LOW, deferred row adds little — fixed as proposed: every row asserts the returned error's class.
- row-validity 13, LOW, env read in `TestStart` — fixed as proposed: the skip flags enter `starter` as a seam.

## forced-edits

- forced-edits 1, BLOCKING, `commitCancelRecord` loses its push — fixed differently: deleting the cancel's push into a direct `PushRun` call would change an out-of-scope path; card 7 instead keeps `commitStatusDeps.Push`, its `push` method and `pushRecordsOrOwe`, deletes only `Admit`, `Owed` and `Sleep`, and stops only the seam and `commitRunRecords` from pushing.
- forced-edits 2, BLOCKING, re-adopt deps unchecked — fixed differently, by the same change: `internal/battencli/readopt.go`'s `CommitFailed` calls `pushRecordsOrOwe`, which stays, so the re-adopt keeps its push.
- forced-edits 3, MEDIUM, unlisted tests — `TestCommitCancelRecord`: fixed by the change above, since its `Push` field stays. `TestFreshStartSeedCommitFailure`: disputed, premise false; `internal/battencli/predrive_test.go` sets a `Push:` field only at line 196, inside `TestFreshStartCommitsWorktreeCreateThroughTheSeam`, so that test builds no deps with a removed field.
- forced-edits 4, MEDIUM, daemon-wakeups background stale — fixed as proposed: card 11 edits `pattern/PATTERN-daemon-wakeups.md`.
- forced-edits 5, MEDIUM, `docs/overview.md` partly covered — fixed as proposed: card 11 rewrites the `internal/statuscommit` line and adds `internal/runpusher/` to the listing.
- forced-edits 6, LOW, stale test comment — fixed as proposed: card 7 names the opening comment of `internal/loomcli/wiring_commitstatus_test.go`.
- forced-edits 7, MEDIUM, anchor change ends pushers — fixed as proposed: card 5 states it and its restart.
- forced-edits 8, LOW, `guardedCommit` callers — fixed as proposed: card 7 names the pull-request record commits.

## guard-reach

- guard-reach 1, BLOCKING, pause checked once per pass — fixed as proposed: every attempt in `PushUntilLanded` reads the pause first, including the push-again after a landing, with `TestRun` rows for a pause during the backoff and after a landing.
- guard-reach 2, MEDIUM, batten resume starts no pair pusher — fixed differently: rather than a start in `lym loom start`'s resume branch, which no test reaches, batten's `startPusher` starts the pair's pusher beside the prime's when the pair stands here, covered by `TestBattenPreRun_ResumeStartsThePusher`.
- guard-reach 3, MEDIUM, prime target with a slug — fixed as proposed: `TargetOf` always gives the prime an empty slug, pinned by a `TestStart` row.
- guard-reach 4, LOW, start behind `noReopen` — fixed as proposed, with a `--no-reopen` row.
- guard-reach 5, LOW (also way-forward 4), owned-elsewhere repeat — fixed as proposed: card 6 states the repeat as a bound that ends at the catch-up's hand-over.
- guard-reach 6, LOW, hand-over during the teardown push — fixed as proposed: card 10 makes it its own Stuck naming the hand-over, terminal, with a row.

## way-forward

- way-forward 1, MEDIUM, pruned tracking ref loops the teardown — fixed as proposed: `TipsOnOrigin` reads an upstream with no tracking ref as held, with a row.
- way-forward 2, MEDIUM, ended records branch halts the teardown — fixed as proposed: card 1 wraps that error in a new `ErrEntryEnded`, and card 10 removes the pair on it without reading `Level`, with rows in both.
- way-forward 3, LOW, Stuck reasons name no next step — fixed as proposed: a refused push's and a failed `Level`'s Stuck carry the error and the resume `lym batten run <slug>`.
