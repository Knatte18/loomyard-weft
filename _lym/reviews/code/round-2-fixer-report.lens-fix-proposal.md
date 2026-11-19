```yaml
lens: fix-proposal
outcome: reported
```

No findings.

**F1 (return the partial outcome on error).** I tried to build a case where it goes wrong and found none:
- The only caller of `diffMutation.run` is `mutationCall.run` (`mutate.go:298`). The verb's error path (`mutate.go:185-191`) prints only the error, so a non-empty outcome on error changes nothing there.
- The report step (`mutation.go:102`) is the one reader of the partial outcome, and `recordFindings` already skips `Functions` and `FileHashes` when `err != nil`, so a partial report cannot match HEAD.
- There is no data race. `judgeMutants` returns only after `workers.Wait()`, so `findingsSoFar` reads `d.runs` and `d.outcome` with no worker running.
- The slot-error return of `d.outcome` is the empty `newMutateOutcome()` that `mutationCall.run` passed in (`mutate.go:296`), as the review asked.
- The partial `counts` can be uneven when `buildLymFor` or `judgeMutants` fails for a package. `Unbuildable` and `NotRun` are counted, but that package's verdicts are not. The report is an error report that never matches HEAD, so no reader acts on that.

**F2 (assert both lists in `TestMutationStep_RecordFindings`).** The proposal is sound.
- The expected `NotJudged` order, pre-check entry first, matches `append(found.notJudged, found.outcome.notJudged...)` at `mutation.go:114`.
- Both lists are asserted in all three rows. Dropping the `Unjudged` assignment, or dropping either half of the `NotJudged` append, now fails the test.

Tool calls: one grep for callers of `run()` and uses of `mutateOutcome{}`. My context held the changed files but not the other `gatecli` files that might call `diffMutation.run`.
