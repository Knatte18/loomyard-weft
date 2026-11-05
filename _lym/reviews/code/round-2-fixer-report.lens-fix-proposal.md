```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal, challenging F1 (`rename-not-done` sets `createNotDone`) and F2 (`exhausted` also requires no unrendered amendment).

**Findings**

1. **MEDIUM** — F1's flag, in the `cardNotDoneFor` case `"create-not-done", "rename-not-done"` (`recordbatch.go`). A rename-not-done finding on the old side now gets a way forward that cannot fix it.
   - The old-side finding (`donecheck.go:430-434`) means "Rename pair's old side %q still resolves — the rename did not happen". The new side is not the problem there.
   - Both sides share the check ID, and the fix sets one flag for both, so the old-side finding now reaches `createTargetWayForward`.
   - Take an old-side finding with `Verb == "recover-batch"`, or a batch that has already been recovered.
     - `recovered` is true, so the result is `amend` alone: "amend the card's Create target or Rename new side to the unit, owner or parameter list the finding names the member declared with…".
     - The old-side finding names no declaration of the new member. Editing the new side leaves the old symbol where it was, and the next recovery fails the same check.
   - Take the same finding on a first `record-batch`.
     - The text offers recover-batch first, then "when the member is declared where the finding names it, amend". That second branch has no referent for this finding.
   - Before the fix, the default `recover-batch` way forward was right for an old-side finding that had not yet been recovered. The change replaced a correct text with a misleading one for that case.
   - The reviewer proposed "one flag covers both". That proposal fails for the old side.
   - The new tests pin only the `renameNotDone` fixture, which has the plain `rename-not-done` check ID. Nothing separates old-side from new-side details, so the gap is unpinned.
   - Pinning the amend text only for the new side would need a distinguishing signal. The finding's `Detail` carries "new side", but `Check` does not.

2. **No findings** on F2. `exhausted := Recoveries >= maxRecoveries && !rendersNewAmendment(in.Batch)` matches `RecoverSpawnOrAttach`'s refusal condition (`recoverbatch.go:509`), minus `!waitsOnParent`.
   - A batch waiting on its parent never reaches `failCardNotDone`, because `RecordBatch` returns at line 373.
   - `failBatch` does not clear `AmendedCards`; the only `= nil` is at `recoverbatch.go:782`. An unrendered entry therefore survives into the terminal record, so recover-batch really would spawn it.

**Tool calls**
- `grep AmendedCards internal/websterengine/*.go` plus `sed -n 425,445p internal/planglyph/donecheck.go`: to see whether `failBatch` retains `AmendedCards` and to read the old-side and new-side finding texts. Neither was in my context.
