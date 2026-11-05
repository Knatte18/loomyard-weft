```yaml
lens: fix-code-correctness
outcome: reported
```

Fork report (lens fix-code-correctness): I found no defect in the changed code, and one test row that pins nothing. I traced each step by hand and did not re-run the gate; the parent's run passed.

- **LOW — `internal/planglyph/donecheck_test.go`, row `GoTargetIsNotAMismatch`.**
  - **Problem:** The row passes whether or not the Go-plan guard exists. A Go glyph never carries parameters, so `wanted.Params == nil` returns before any comparison. The row's declarations also hold no `Thing` member, so nothing could match anyway.
  - **Gap:** Review finding F4 asked for a Go-target negative row, and this one never reaches the guard it names.
  - **Fix:** Add a declaration of the same name under the same unit that does not equal the key. A Go glyph has no parameter list, so this can't happen for a not_found key. The row is therefore inherently weak, and the finding is low-value.

What I traced and found correct:
- **`createTargetWayForward`:**
  - Order is uncheckable, then exhausted, then recovered/signatureMismatch, then both routes.
  - `TrimPrefix` of `"way forward: "` leaves exactly the clause that `BatchFailedError.Error` re-prefixes.
  - `failCardNotDone` is the only caller, and it passes the new fifth argument.
- **`RecoverSpawnOrAttach`:** The message is byte-identical to before. The old `"%s; or %s"` is now inside `exhaustedRecoveryWayForward`, and the prefix still comes from `wayForwardSteps`.
- **`TestFailCardNotDone_WayForward`:** Each row would fail with its guard removed.
  - Dropping the `signature-mismatch` case in `cardNotDoneFor` makes `SignatureMismatchAmendsAlone` return the default way forward instead of `amend`.
  - Moving the create arm before the glyph arm, or before the delete arm, fails `GlyphRejectedBeatsCreateNotDone` and `ReferencedDeleteBeatsCreateNotDone`.
  - Dropping `exhausted` fails the two exhausted rows.
  - `UnreferencedDeleteLeavesCreateNotDone` pins the `len(referenced) > 0` guard.
  - The fake `Index` embeds a nil interface, which is safe because only `LaterDeleteReferences` is reached, and only when a delete flag is set.
  - `CardNotDoneFor` returns a true nil error for empty findings, and `errors.As` finds `*cardNotDoneError` otherwise.
- **`donecheck_test.go` rows:** The Rename new-side row fails without the `rename-not-done-new` branch in `signatureMismatchFinding`. The no-parameter-list, other-owner and other-unit rows fail if their guard in `signatureMismatches` is dropped. The exact-equality details pass in the parent's gate run.
- **Behaviour of `Recoveries >= maxRecoveries` at failure time:** A failure after the second counted spawn (`Recoveries == 2`) takes the exhausted steps, and a failure after the first (`Recoveries == 1`) takes the amend route. A fork batch re-begun with carried-over `Recoveries == 2` that fails create-not-done in record-batch also gets the exhausted steps, which matches what `RecoverSpawnOrAttach` would say for it. Nothing in the changed code contradicts this.

Tool calls: one `grep` for every call site of `createTargetWayForward`, `exhaustedRecoveryWayForward` and every `Recoveries` write. My context lacked the caller set and the places that mutate `Recoveries`.
