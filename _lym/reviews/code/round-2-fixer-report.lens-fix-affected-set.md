Lens fix-affected-set: I checked F15, F16 and F17 for members the fix missed and found none.

no findings

- **F15:** `b.settle` and `settleUnlessRefused` are called in non-test code only at `bouncer.go:453`, `477`, `526`, `528` and `2099`. Every call except `526` goes through `settleUnlessRefused`, which now settles the ask and applies the parent-findings refusal. `526` is the `standsAsRecorded` replay. It skips the refusal on purpose, since the parent's decision settles over that round, and the old code skipped it too. `bouncerdecision.go` has no other settle caller.
- **F16:** no `judgeParentBlock` remains in `internal` or `contracts`. The only other embedded prompt text my grep touched is `parent-review await` in CLI help and doc comments, which is not prompt text.
- **F17:** `p.judgeStartedAt` is now set only at `verify.go:886`, in `DriftVerify.runAgent`. Both callers (`verify.go:476` for line mode, `1072` for command mode) pass through it. The `runAgent` at `expect.go:147` belongs to `DriftExpect` and has no parent ask.

Tool calls:
1. One grep for `b.settle`, `settleUnlessRefused`, `parentFindingsRefusal`, `heldBeforeJudgeAsk` and `beforeJudgeAsk` in `internal/shedadapters`. It re-ran F15's set of settle paths, which my context did not list in full.
2. The same call also ran the F16 remnant grep for `judgeParentBlock` and `parent-review await` across `internal` and `contracts`.
3. The same call also ran the F17 grep for `judgeStartedAt`, `runAgent` and `judgeAskLimit` in `internal/driftengine`.
