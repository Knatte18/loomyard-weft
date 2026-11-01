```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

The ending side of the lost-writer branch, `resume.go:129-134`, has two ways in: `allExist(outputs)` is false, or `lost.accept` returns an error. The fix has one subtest for each:

- **Missing outputs:** no output file is written, and `acceptCalls` is asserted to be 0. This also checks that the `&&` short-circuit skips `Accept`.
- **Accept error:** the output is written and `Accept` returns an error. The subtest asserts `EndedStep` and `errors.Is(AcceptErr, rejected)`.

Both subtests also assert empty `StrandGUID` and `Outcome`, one chair stop and no line sent to the chair.

The other `ender.end` sites are outside F1's premise. `resume.go:136` is the lost required advisor. `engine.go:143` and `engine.go:268` are the fresh-run watch and start-failure paths. All three have their own existing tests: `TestEngine_Run_RequiredAdvisorEndsTheStep`, `TestEngine_Run_WriterEndDecidesTheJoin` and the start-error rows.

Unchecked: the subtests use a single-output table, so "some but not all outputs exist" is not exercised. `allExist` is shared and covered by its own semantics, and the review did not ask for that case.

Tool calls:
1. Ran `grep` for `ender.end(` and `allExist(` in `internal/seatengine/*.go`. The files I had read were only `resume.go` and `engine.go`, so I needed the full site list for the set.
