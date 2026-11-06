```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens, running each changed function and branch on a concrete input.

**Per-branch list**

- `BeginBatch`, `orderOnly` accumulation (`beginbatch.go:497-503`): correct.
  - Input: no findings. The blocking branch is never entered, so the initial `true` is never read and the cause is unused.
  - Input: only informational findings. The `else` arm skips the `orderOnly` update, so an informational `delete-before-reference` cannot set it. `blocking` stays empty and `BeginBatch` doesn't refuse.
  - Input: one blocking `delete-before-reference` finding. `orderOnly` stays `true`.
  - Input: blocking `delete-before-reference` plus blocking `glyph-not-found`, in either order. `orderOnly` goes `false` and stays false, because `&&` is monotone, so the order doesn't matter.
  - Input: informational `delete-before-reference` plus a blocking `glyph-not-found`. Only the blocking finding is counted, `orderOnly` is `false`, and the cause is the old one. Correct.
- `BeginBatch`, cause `switch` (`beginbatch.go:511-518`): correct.
  - Input: `orderOnly` with `begun` empty gives `plan_order`.
  - Input: `orderOnly` with `begun` non-empty gives `plan_order`.
  - Input: not `orderOnly` with `begun` empty gives `plan_defect`.
  - Input: not `orderOnly` with `begun` non-empty gives the default, `tree_moved`.
  - Because `orderOnly` is only read when `len(blocking) > 0`, `plan_order` can't be produced with zero findings. `Findings` is always non-empty, so the conflict record's finding lines and `ClaimConflictRoute` behave as before.
- `PlanConflictError.Error` (`beginbatch.go:113-120`): correct.
  - Input: `plan_order` renders "move each named delete to a card after the card that still references it, run "lym webster rebaseline --card NN" naming each card you edited, then begin-batch NN again".
  - Input: any other cause, including an empty `Cause`, renders the old text byte for byte, apart from the `%s` substitution.
  - The rebaseline and begin-batch tail is intact. `Unwrap` and `errors.Is(ErrPlanConflict)` are unchanged.
- `WriteConflictRecord` (`conflictrecord.go:42`) and the plan-fixer path:
  - `cause: plan_order` is written verbatim.
  - `ReadConflictRecord`, `ClaimConflictRoute` and `planfixer.go` (`FixPlanSpawnOrAttach`, lines 94-104) read only the batch number and the finding lines, never the cause. No switch or validation rejects `plan_order`, so a `plan_order` conflict is accepted wherever the old two causes were.
- `planglyph/deleteorder.go`, three emit sites (lines 260, 294, 375): correct. `planindex.CheckDeleteBeforeReference` is the same string as the old literal, so emitted findings are identical. `planglyph` already imported `planindex` through the shared `Finding` type, so there is no new import cycle.
  - Residual risk, not a defect: the string comparison in `internal/planglyph` tests (`deleteorder_test.go:240`, `planfixture_helper_test.go:106`) still uses the literal. That is test code and harmless.
- `dispatchFindingsIndex.ValidateDispatch` in the new test: it embeds `planindex.Index` and overrides only `ValidateDispatch`. The `BeginBatch` re-resolve uses that method for its findings, so the stub drives exactly the branch under test.

**Findings**

- LOW, `design`, `internal/websterengine/beginbatch.go:503`: a `plan_order` refusal fires even when the finding's deleting card is already begun. On the `begun` path the way forward says to move the delete, but a begun card's delete can't be moved. I couldn't confirm whether `ValidateDispatch` ever reports a delete from a begun, non-terminal card. If it can, the cause and way forward mislead in that corner. Evidence: `beginbatch.go:497-519`, with the `ValidateDispatch` semantics in the `planglyph` docs (`doc.go`, "pending cards"). I'm reporting it as a possibility, not a confirmed defect.

No other defects found in the changed code.

**Tool calls**

I made no tool calls; the context already held every file this lens needed.
