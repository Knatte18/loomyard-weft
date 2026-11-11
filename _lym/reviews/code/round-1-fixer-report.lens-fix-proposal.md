```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F1, F2, F4–F8), from the context I already hold. I made no further tool calls.

**MEDIUM — F4, `contracts/specs/loom-discussion-spec.md`, new sentence after the replace bullet**
- **Claim at issue:** the sentence says the stored record on a discussed board entry "is folded with `lym board upsert --record-file`, not with `replace`".
- **Why it is wrong:** `TestDecisionCalls_RefreshEveryStoredRecord` has the row "ReplaceOnARaddleRunOfAPromotedSketch", where `replace` is admitted and copies the committed record onto the entry. The `FiledRaddleRun` refusal also sends the caller to `decision replace <part-slug>`. So `replace` does change the stored copy on such runs.
- **What I could not check:** the review's fix wording only asked the spec to name `--record-file` and say when each verb is used. The "not with `replace`" half, and the "once a motif run has made it" premise, are the fixer's own additions. I found nothing in the code I hold that supports either.
- **Suggested change:** drop "not with `replace`" and state the case `--record-file` serves, for example a stored record with no run to replace on.

**LOW — F5, the `replace` help text and `replaceFileRefusal` doc, left unchanged**
- **What they say:** the help text says the file may hold no added entry "the record does not hold byte for byte". The `replaceFileRefusal` doc says each entry must "equal one of current's".
- **What the fix keeps:** `CheckAddedEntriesSubset` still compares through `addedDecisionSpanText`, which trims trailing whitespace. Only the function's own doc comment was updated to say so.
- **Case:** an entry whose last line differs only by trailing spaces passes the check, against the "byte for byte" wording.
- **Why it matters:** the review offered "state in the doc comment" as an option, so this is acceptable, but the help text still contradicts it.

**LOW — F8, `recordhistory_test.go`, `len(history) < len(whole)`**
- **Fragile check:** this infers "cut" from length. A cut at the oldest version returns every version, so it would be read as uncut and trip `wantCut: true`.
- **Current rows:** none hit it, so nothing fails today.
- **Test setup:** `whole` is built from the first-parent versions, while production's `readWhole` is the committed path history. The fallback is therefore only checked as "return what `readWhole` gives", which is the intended contract.

I found no breakage in the F1, F2 or F7 proposals. The `committedAfterErr` condition change in the fake (`replaceCommits > 0`) only matters to rows that set that field.

Calls made: none.
