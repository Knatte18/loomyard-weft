```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

**F4 and F2 set:** I ran `grep -rnE "AnswerParentAsk([^OT]|$)"` over the repo, leaving out `.lym`, `_lym/reviews`, `.git` and `.scratch`. No function reference or `AnswerParentAsk()` message is left in Go code.
- What remains is `TestAnswerParentAsk`, `TestAnswerParentAsk_FailedWriteAtDeadline` and `TestAnswerParentAsk_RacingTheDeadline`. The review said to keep those test names.
- It also finds the `//testtiming:keep` markers in `internal/burlerengine/template_test.go:99` and `:192`, which cite `TestAnswerParentAsk` by test name.
- The rest are `docs/research/speed/...` data files. They are dated measurement reports, so they are expected to go stale.

**F1 set:** I searched `contracts` and `docs` for the no-parent wording. The only hits are the reworded `role-autonomy.md:48` and the stencil lines `landing-block-pr-review-autonomous.md:29` and `:31`. The stencil says to run `await` first "after a no-parent or undelivered end". That agrees with the new wording, and the stencil was left unchanged as the review specified.

**Unchecked:** F3 and F5 are single-site additions, not sets, so I did not search them.

Tool calls:
1. `grep` for `AnswerParentAsk` not followed by `O` or `T`, repo-wide. My context only held the review's line list, so I needed the full set.
2. `grep` for the no-parent wording in `contracts` and `docs`. My context did not show any other copy of the sentence.
