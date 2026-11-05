---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: internal/websterengine/recordbatch.go:920
    summary: a Rename new side quarry answers not_found is check rename-not-done, which cardNotDoneFor does not flag, so it keeps the recover-batch default way forward the diff claims to replace
    fix: In cardNotDoneFor's switch on f.Check, add a case for "rename-not-done" that sets createNotDone, so failCardNotDone offers createTargetWayForward for it. Only the new side can need the amendment, but its old-side finding has the same check ID and the same recovery loop, so one flag covers both. Then update the createTargetWayForward and failCardNotDone doc comments, the doc.go sentence in internal/websterengine, and the summary sentence so they say the way forward covers a rename-not-done finding. Add a row to TestFailCardNotDone_WayForward with a "rename-not-done" finding on record-batch that wants the recover-batch-or-amend text, and one on an exhausted batch that wants the exhausted steps.
  - id: F2
    severity: LOW
    class: design
    location: internal/websterengine/recordbatch.go:227
    summary: exhausted is true by Recoveries alone, but RecoverSpawnOrAttach still spawns an exhausted batch whose record holds an unrendered amendment, so the exhausted steps can be named where the amendment route works
    fix: In failCardNotDone, compute exhausted as in.Batch.Recoveries >= maxRecoveries && !rendersNewAmendment(in.Batch), the condition RecoverSpawnOrAttach uses for its ErrRecoveryExhausted refusal. Add a TestFailCardNotDone_WayForward row with Recoveries at MaxRecoveries and a BatchState.AmendedCards entry with Rendered false that wants the amendment text.
---

### [MEDIUM] Rename new side keeps the self-pointing way forward

**Location:** internal/websterengine/recordbatch.go:920
**Issue:** `doneCheckVerdicts` reports a Rename pair's new side that quarry answers `not_found` under the check `rename-not-done`, with the old side under the same ID (`internal/planglyph/donecheck.go:434-439`). `cardNotDoneFor` sets `createNotDone` only for `create-not-done`, and `signatureMismatch` only for `signature-mismatch`. A plain missing Rename new side therefore sets no flag, `failCardNotDone` leaves `wayForward` empty, and `failBatch` falls back to its default, `lym webster recover-batch NN`. That is the dead end the issue describes, and for an exhausted batch it ends in `ErrRecoveryExhausted`. The text of `createTargetWayForward` and the change description both tell the reader to amend the "Rename new side", but only a signature-mismatch finding on a Rename new side reaches it. The change description's claim that the way forward covers a Create target or Rename new side is therefore not borne out for the plain not-found case.
Round 1 deferred this as predating the change, but the diff itself now words the way forward for the Rename new side and the summary claims it, so the gap is the diff's own.

### [LOW] Exhausted steps shadow a pending amendment

**Location:** internal/websterengine/recordbatch.go:227
**Issue:** `RecoverSpawnOrAttach` refuses with `ErrRecoveryExhausted` only when `prior.Recoveries >= maxRecoveries && !rendersNewAmendment(prior) && !waitsOnParent(prior)`. `failCardNotDone` tests `Recoveries` alone. A batch at the cap whose in-flight recovery was amended through `Rebaseline` has an unrendered `AmendedCards` entry. If that recovery then fails a done-check, the diff sends the reader to reset or restart although `recover-batch` would spawn and render the amendment. The window is narrow, since `Rebaseline` records an amendment only for a non-terminal batch, but the two conditions should agree.

I checked the first focus question against `Rebaseline` and `RecoverSpawnOrAttach`. For a terminal failed batch `Rebaseline` records no amendment (`rebaseline.go:374`), so the exhausted batch's amend route would end in the refusal, and the diff correctly sends it to the exhausted steps. The tests reach the signature-mismatch way forward, the `cardNotDoneFor` flag, the uncheckable arm, the exhausted case and the precedence of the delete, glyph and create ways forward. The `donecheck_test.go` rows pin the no-mismatch guards and the Rename new side wording.
