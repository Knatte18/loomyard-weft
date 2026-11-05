```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens over the F1–F5 change set.

**F-A (LOW): the glyph-rejected and Delete-reference way forwards keep the dead end F1 removed from the Create one.**
- Changed text: `createTargetWayForward` now takes an `exhausted` flag. `glyphRejectedWayForward` (`internal/websterengine/recordbatch.go:128`) and `deleteReferencedWayForward` (`:119`) were left as they were.
- Evidence: both still end in `... rebaseline --card NN ... then lym webster recover-batch NN`.
- Evidence: F1's premise is that a failed batch records no amendment, because `Rebaseline` skips a terminal batch. After the second counted recovery, that route ends in the `ErrRecoveryExhausted` refusal at `recoverbatch.go:509`.
- Evidence: that premise does not depend on the target kind, and `failCardNotDone` has `in.Batch.Recoveries` available for all three arms.
- Why it is only LOW: the review scoped F1 to Create targets, so this is an unfixed twin rather than a regression.
- Related gap: `failCardNotDone`'s doc comment now describes the exhausted case for Create only.

**F-B (LOW): `_lym/landing/summary.md` still says "amend the card's Create target" in its first way-forward bullet.**
- Evidence: the first bullet under "Way forward for a missing Create target" reads "amend the card's Create target, run `lym webster rebaseline --card NN`, then `recover-batch`".
- Evidence: F3 changed the code text to "the card's Create target or Rename new side".
- Evidence: the summary's "What changed" paragraph for `signature-mismatch` also says only "Create target". The code now covers a Rename new side too.
- I did not re-read `summary.md`; this is from the line 24 text in the context above and the unchanged bullet in your edit's `old_string` neighbourhood.

**Checked, no finding:**
- `createTargetWayForward` has one caller, and `exhaustedRecoveryWayForward` has two. The exhausted text matches the old `RecoverSpawnOrAttach` message exactly.
- The `RecoverSpawnOrAttach` format string is `"%w: batch %02d has had %d recoveries; %s"`, with the way forward now coming whole from the helper. It produces the same output as before.
- `doc.go:449` ("an amendment forces at most one uncounted re-run") is unchanged and still true for an in-flight batch. The summary sentences that contradicted it were the ones deleted.
- The existing test substring "amend the card's Create target" still matches the reworded text as a prefix.
- The gate passed with 0 failures.

**Tool calls (2 of 6):**
1. `grep` for "amend the card's", "uncounted" and "Create target or Rename" across `.md`, `.go` and `.yaml` outside tests and the review. I needed it because my context lacked docs or comments that mention the changed text or the uncounted-recovery claim.
2. `grep` for `Recoveries++`, `Recoveries =` and `Recoveries+=` in non-test `websterengine` files. I needed it to see where the counter is incremented, because that sets what `Batch.Recoveries` holds when `failCardNotDone` runs. It showed only `beginbatch.go:641` and `maxRecoveries`, so the increment point is not settled by this grep and stays unverified.
