```yaml
lens: fix-consistency
outcome: reported
```

Task restated: check the fix set for stale twins and cross-reference disagreements.

Findings:

1. **LOW**, `internal/loomrecipe/doc.go:55`, `internal/fabricengine/replay.go:250` and `internal/fabricengine/doc.go:1761`.
   - These still say Raddle-File "ends the run's entry" with no family exception. They are twins of the sentences the fix reworded in `docs/overview.md:1302-1303`, `contracts/recipes/raddle-recipe.yaml:4`, `internal/loomcli/wiring.go:232` and `internal/loomshed/raddlefile.go:4`.
   - The `loomrecipe/doc.go` line states the same claim as the `raddle-recipe.yaml:4` comment that F9 fixed, in the same words, and was left unchanged.
   - The two `fabricengine` lines describe reconcile logic (an origin showing the entry done is the run's own finish). A family run's entry never reaches done, so the "entry origin shows done" premise does not hold for it. That is a possible behaviour gap, not just wording, and I did not check it.
   - Evidence: the grep output above shows the unchanged sentences beside the updated ones.

2. **LOW**, `internal/loomshed/raddlefile.go:139`.
   - The `Call` doc says "step 6 ends the run's entry, leaving the row Done". The next line, "A family filing skips step 6", does state the exception, so the two lines agree. No change is needed.

3. **LOW**, `internal/boardengine/runlock.go:211`.
   - The `scopeChanged` comment says `RunSettings`, `Profile` and `NotifyOnLand` are "never compared". That is still true of `scopeChanged`.
   - `endsAsFamilyProject` now compares them, and the `checkRunLocks` doc at `runlock.go:146-148` says only "exactly the project the entry and the added parts derive". It does not say these three fields must be clear.
   - The `endsAsFamilyProject` doc was updated, so the lock's own documentation is consistent. This is cosmetic.

4. **No finding**, `internal/boardengine/raddlefile.go:2`.
   - The file header says the file "ends the run's entry", which is true of `EndRaddleEntry`. The line below it carries the family exception ("A family filing turns the run's entry into the family's project instead"). Consistent.

5. **No finding**, other cross-references.
   - `loomcli/doc.go:66`, `loomshed/doc.go:266`, `boardengine/doc.go:636` and `battencli/commitstatus.go:124` already carry the family exception.
   - The lock rows and the F6 test names agree with `endsAsFamilyProject` as changed.
   - `RaddleFileProducer` is still pinned by the existing loomshed test.

Tool calls (2):
- `grep` for "ends the run's own entry" and similar phrasings across `.go`, `.md` and `.yaml` files: to find sibling statements of the claim F8 and F9 corrected, which my context lacked.
- `grep` for "family filing", `endsAsFamilyProject` and the not-compared fields comment: to check that the docs around the lock and the filing agree, which my context lacked.
