```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fixer's change set for code correctness (the new `TestAnswerParentAsk` row, the renamed messages, the keep marker, the contract and summary text).

No findings.

- **New row** "a no-parent ask with no tag is refused when the call names no unreached tag" (F5): the ask has an empty `Tag` and `unreachedTag` is empty.
  - With the guard `unreachedTag != ""` in place (`internal/burlerengine/parentreview.go:371`), `unreached` is false. The ask is `NoParent`, which `Ended()` covers, so the call returns a `ParentAnswerEnded` refusal. The state stays `[no-parent]` and no report is written, which matches the row.
  - With the guard deleted, `latest.Tag == unreachedTag` becomes `"" == ""`, so `unreached` is true. The ask is answered and set done, so the row fails on both the refusal check and the state check. The row therefore pins the guard.
- **Message and comment renames** (F4): only the string and comment text changed, and no signature or behavior moved. `lym gate test` passed with 0 failures, as the parent ran it. Function names such as `TestAnswerParentAsk` were left alone, as the review asked.
- **Keep marker** (F3): `lym loom fmt` placed it after a `//` line below the doc comment, which is the form the other keep markers in the repository take. This is a comment-only change.
- **Callers:** no signature or behavior changed in this round, so no caller is affected.

Calls made:
1. Read `internal/burlerengine/parentreview.go` lines 355–384, because my context lacked the `unreached` guard and refusal switch that the new row exercises.
