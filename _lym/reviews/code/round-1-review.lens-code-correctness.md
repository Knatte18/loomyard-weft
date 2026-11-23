```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff (`AnswerParentAskOrUnreached`, its callers and the answer verb).

**Cases run**

- `AnswerParentAskOrUnreached`, empty record: `ok` is false, so `unreached` is false and the result is a `no-ask` refusal with nothing written. Correct.
- Same function, number ≠ latest, where the older ask is no-parent under the tag: refused `not-latest` before the unreached path is used. Correct.
- Latest ask no-parent, tag matches, not cleared, zero `Deadline`: `unreached` is true and the ended case is skipped. The deadline check is skipped too, so the report is written and the ask becomes done with `Report` and `AnsweredAt` set. Correct.
- Latest ask undelivered under the tag, past its deadline: the timeout branch is skipped because `!unreached` is false, so it is answered done. Correct.
- Latest ask no-parent under the tag with `Cleared` set: `unreached` is false, so it is refused `ended` and stays unchanged. Correct.
- Latest ask no-parent, call passes `unreachedTag == ""`: the `""` guard holds, so it is refused `ended`. Correct. This also covers every non-PR-Review caller.
- Latest ask no-parent under another tag (a code-review tag): refused `ended`. Correct.
- Latest ask done or timeout under the tag: `unreached` is false, so it is refused `ended`. A done ask cannot be overwritten. Correct.
- Unreached ask whose `writeParentReport` fails (unwritable dir): the closure returns the error, `UpdateJSON` writes nothing, and the call returns a plain wrapped error. The ask stays no-parent and nothing is half-written. Correct.
- Answer racing `ExpireParentAsk`, `ClearParentAsk` or a new `RecordParentAsk`: the check and the write happen in one `state.UpdateJSON` under `files.Lock`. Expire is a no-op on an ended ask. Clear or a new ask either lands first, which makes this call refuse (`ended` or `not-latest`), or lands after the done state. So only one end state can result. Correct.
- Pending-post or open ask before its deadline, with the tag passed: `unreached` is false and the original path answers it. That path is unchanged. Correct.
- `parentReviewAnswerVerb`, no-parent PR ask, caller is the parent orch: `asked.Strand` is `""`, the caller check passes on `parent.Name`, and the answer is recorded done. The success output is the same shape as for a live answer.
- Same verb, parent resolution failing at answer time, caller is an orch strand: refused by `parentAnswerCallerRefusal`, so only the operator's unnamed shell can answer. Consistent with an ask that never resolved a parent.
- Same verb, cleared no-parent PR ask: refusal text reads "ask 1 is already no-parent; way forward: … [pr-review request] line". Correct.
- Downstream of a satisfied answer to an unreached ask at head H: `ReadPRReviewStanding` (`LatestTagged`, `Digest == H`, done, empty findings) returns satisfied. `pr-continue` (`prcontinue.go:106`) and the row (`prreview.go:979`) then proceed. Correct.
- `parentReviewAwaitVerb` after such an answer: `ExpireParentAsk` is a no-op and the verb reports `ready true`, `done`, `report`. This matches the stencil's "run `await` first". Correct.

**Findings**

- NIT, design. Location: `internal/burlerengine/parentreview.go:394`. An ask answered through the unreached path becomes done but keeps the `Reason` its no-parent or undelivered end wrote, so the record holds a done ask with a stale failure reason. Evidence: line 394 sets only `State`, `Report` and `AnsweredAt`. `Reason` was set at `internal/landingshed/prreviewask.go:52-54` and `:64`. No reader shows `Reason` for a done ask (`internal/loomcli/parentreview.go:274-279`), so nothing goes wrong today.

No other correctness defects found.

Tool calls:
1. `loomcli/parentreview.go` 330–365, plus the callers of `AnsweredAt` and `ClearParentAsk`. The manifest's read stopped at line 330, and I needed to know who consumes or clears a done answered ask.
2. Grep for the consumers of `ReadPRReviewStanding`, `PRReviewSatisfied` and the asker. I needed to confirm that `pr-continue` and the row accept a satisfied answer at the head.
