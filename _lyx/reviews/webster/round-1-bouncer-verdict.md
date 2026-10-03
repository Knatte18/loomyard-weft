---
verdict: BLOCKING
rationale: "F1-F5 are fixed and verified (go build, go vet, go test ./..., integration and gofmt -l all clean), but the comment-convention defect F3 named persists in the diff's _test.go files, where new and changed Go comments (bouncer_verdict_test.go, bouncer_circling_test.go, entries_bouncer_test.go, review_test.go, config_test.go and the vocabulary swaps in bouncer_clear_test.go and bouncer_commit_test.go among others) are still fixed-column wrapped mid-sentence against CONSTRAINTS.md's Comment Line-Break Convention"
---

# Webster-Bouncer — round 1 judgment

## The review

The review range is correct.
`_lyx/shed/review-finding-class/status.json` gives `product.parent` as `main`, and `_lyx/loom/rework/` does not exist, so the range is `git diff $(git merge-base main HEAD)..HEAD` over merge base `5805c58fb`.
All five findings hold against the diff, and the focus-directive verdicts agree with what I checked.
The report has no `## Focus departures` section, so there is nothing to ratify.

## The fixes

Each fix is in its own commit and checks out:

- F1 (`273745f04`): `SegmentBounces` is in `nilLegal`, and `go test ./...` passes with no failing package.
- F2 (`f9b0a9288`): `gofmt -l` over every changed Go file is empty.
- F3 (`8e7defc94`): the listed non-test comments are one sentence per line.
  The `-- seed, re-bounce, judge, or replay --` break in `internal/shedadapters/doc.go` falls at an aside boundary, which is acceptable.
- F4 (`cb3cd41ee`): `circlingStatusReader` decides absence from the status file and creates the missing lock directory, and `TestCirclingStatusReader` covers both cases.
- F5 (`4b3f38814`): the Review Round Invariant list is tight.

`go test -tags integration` passes for `internal/loomcli` and `internal/shedadapters`.
Card 11's deleted design doc has no remaining reference outside `_lyx`.

## Why the round does not converge

CONSTRAINTS.md's Comment Line-Break Convention covers all Go comments, doc and inline alike, and test files are no exception.
The review scoped F3 to non-test files, and the fixer fixed only that scope.
New and changed comments in the diff's `_test.go` files are still wrapped at a fixed column, with breaks mid-phrase:

- New comments, newly fixed-column wrapped: the file headers of `internal/shedadapters/bouncer_verdict_test.go` and `bouncer_circling_test.go`, `writeFactsReview` in `bouncerfacts_test.go`, `TestBouncerEntry_EnvJudgeFallback` in `internal/shedrecipe/entries_bouncer_test.go`, `TestResolveJudge` and `TestResolveJudge_MalformedSpec` in `internal/loomengine/review_test.go`, `TestLoadConfig_MalformedJudgeSpec` in `internal/loomengine/config_test.go`, and the rewrapped header of `internal/burlerengine/verdict_test.go`.
- Changed lines inside legacy wrapped sentences, where the sentence straddles the changed line: the `APPROVED`→`CONVERGED` and `BLOCKING`→`CONTINUE` swaps in `internal/shedadapters/bouncer_clear_test.go`, `bouncer_commit_test.go`, `bouncer_replay_test.go`, `bouncer_judge_test.go`, `burler_test.go` (`writeJudgedRound`), and `internal/loomrecipe/fixture_test.go`, `resume_test.go`, `sequence_test.go`.

The fixer reflowed sentences that straddle a changed line in the non-test files, and the same rule applies here.
The defect is the one F3 named, so the diff still fails a dimension this rubric flags.
