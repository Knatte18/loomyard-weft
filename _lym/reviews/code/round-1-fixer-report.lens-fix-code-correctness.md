```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: I ran the changed branches step by step against concrete inputs and checked that the added tests fail with the defect in place.

**Branches traced, no defect found**
- `MergeState.MergeabilityUncomputed` (`internal/pullrequest/mergestate.go`) is true only for an open pull request with `MergeableState` `unknown` and exactly one cause. `mergeCauses` adds the uncomputed cause alone for an open state, and a draft or a head mismatch adds a second cause, so `len(Causes)==1` means that cause is the only one. A closed or merged state is excluded by the `StateOpen` test.
- `mergeRefusalReason`: when `state.MergeableNow` is false and the state is uncomputed, it returns the wait text with the resume. The `MergeableNow` early return runs first, so a state that can merge never reaches the new branch. A method refusal never carries a state, so the method branches are unaffected. A beam adds ", which lym beam pr status <slug> shows".
- F1 row: with the old text in place, the reason would hold "approves the pull request again" and "lym batten cancel", so the added `wantNotInReason` entries fail it.
- F2 row, `Status` scripted `{open, {err: 404}}` with `Merge` `{notMergeable}`:
  - The merge is refused as `RefusalOther`, so `ReadMergeState` runs and its first `api.Status` fails before `RequiredChecks`.
  - The error is wrapped with "pullrequest: read pull request #7: …" and reaches `githubFailure` with action "read whether the pull request can merge".
  - The same non-transient `errors.New("404 Not Found")` halts in the existing row at line 352, so it halts here.
  - Calls are Status, merge, Status, with no second merge. Without the `return` after the error, the loop would continue to `RequiredChecks`, so the call list would differ.
- The signature of `mergeRefusalReason` is unchanged, and `MergeabilityUncomputed` is new and exported with no other callers.

**Findings**
- LOW: `internal/landingshed/finalize_pr.go`, the `mergeOnGitHub` doc line "the merge is retried while that state reports the pull request can merge, scheduled only while the window has not passed." The text after `//` is about 121 characters, over the 120-character limit the prose skill sets. Split it at the comma before "scheduled".
- LOW: the new beam wait text in `mergeRefusalReason` (", which lym beam pr status %s shows") has no test row. The beam rows cover only a `blocked` state. This is a coverage gap in code the fix added, not a wrong result. `fz.deps.Slug` is the same source the existing beam text uses.

Further tool calls: none. My context already held every changed file.
