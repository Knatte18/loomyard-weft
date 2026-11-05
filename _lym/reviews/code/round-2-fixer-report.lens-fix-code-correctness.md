```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the F1/F2 change set (`cardNotDoneFor`, `failCardNotDone`, `createTargetWayForward` and the new test rows).

**LOW — `recordbatch.go` `cardNotDoneFor` case `"create-not-done", "rename-not-done"`: the old-side finding gets create-target wording.**
- `donecheck.go:434` reports a Rename old side that still resolves under `rename-not-done`, and the same flag now covers it.
- For that finding the way forward says to amend "the card's Create target or Rename new side to the unit … the finding names the member declared with". The old-side detail names no declaration site, so the amend clause does not fit it.
- After a recovery (`Verb == "recover-batch"`) the amendment comes alone, so `recover-batch` is not offered even though a plain retry of the rename is what the old-side finding calls for.
- The review's fix text accepts one flag for both sides, so this is a wording mismatch and not a wrong result.

No other findings. I ran each branch by hand:
- **`rename-not-done` on the new side:** `createNotDone` is set, so `createTargetWayForward(..., signatureMismatch=false, exhausted=false)` runs. On `record-batch` it returns the recover-or-amend text, and on `recover-batch` the amend text alone. This matches the new test rows.
- **`exhausted` with an unrendered amendment:** `Recoveries` at the cap with an `AmendedCards` entry of `Rendered: false` makes `rendersNewAmendment` true, so `exhausted` is false. With `Verb` `recover-batch` the result is the amend text, and the new row wants exactly that.
- **`exhausted` with no amendment:** a nil `AmendedCards` makes `rendersNewAmendment` false, so `exhausted` stays true. The existing exhausted rows hold.
- **Nil safety:** `in.Batch` is always non-nil in `failCardNotDone`, since it is dereferenced on the previous line.

Each new row fails with its defect in place:
- **`RenameNotDoneOffersRecoverAndAmend`:** with no flag set, `wayForward` is empty and `failBatch` falls back to its default.
- **`ExhaustedRecoveriesBeatRenameNotDone`:** without the flag it also gets the default.
- **`ExhaustedBatchWithAnUnrenderedAmendmentAmendsAlone`:** with `exhausted` computed from `Recoveries` alone, it gets the exhausted steps.

The test struct gained a trailing `amended` field and every existing row got a trailing `nil`, so no positional literal is left short. The gate passed, per the parent's run.

Tool calls: none; my context already held every changed file, the test and the gate result.
