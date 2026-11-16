```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fixer's change set for agreement with sibling cards and cross-references, and for twins left unchanged (fix-consistency lens).

Three findings.

1. **MEDIUM**: card 11, the new `planVerifySites` bullet ("gains the weaverengine file that holds `verifytree.Verify` as `RunDeps.Verify`'s production value with the `Hold` it passes").
   The bullet may name a file that does not exist or does not pass the tripwire at card 11's gate.
   - `cmd/lym/verifiedtree_test.go:68-77` requires each listed site to hold both `verifytree.Verify` and `Hold:`/`.Hold =`.
   - Lines 81-84 of the same test fail on a listed file it does not find.
   - Card 11 says `Run` refuses a nil seam. `Run` is card 12's, so the nil-to-`verifytree.Verify` default may land in card 12's file, while card 11's `runVerify` file passes the `Hold`.
   - Card 11 does not fix which card-11 file holds both tokens. The fix should name the file, or say the default and the `Hold` sit in `runVerify`'s own file.

2. **LOW**: card 5's `StopRecord` description ("`Stop *StopRecord`, the row's last batch, needs-parent or verify stop"; "which D7's re-entry reads") is now stale.
   - Card 12 now clears `StopRecord` at each batch begin.
   - Card 13 now clears it once the stopped batch is recorded done or the verify re-run passes.
   - Card 5 still describes the field as the row's last stop, with no mention that it is cleared.

3. **LOW**: `00-overview.md:38`, the card-index line for card 22 ("Describe and the PR rework archive cover Weaver's run directory"), was not updated.
   Card 22 now also moves `failuretrace.NewTracer`, `Input`, `Trace` and `lastCardSHA` onto the seam and `runplan.State`, as its Intent and ImpactSummary say. The index line does not mention the failure tracer.

Tool calls:
- grep of the plan cards for the removed tests, `ConflictRecordRel`, `StopRecord`, `saveRunState`, the reset marker, `failuretrace` and the lease. My context did not hold the cards the fixer never opened (5, 6, 7, 15 to 19, 24 to 26).
- grep of cards 15 and 26 and the other cards for the pending-commit, `heldBackReason`, `verify_gate` and `Setting` references. This checked the twins of the card 14, 21 and 22 changes in cards outside my context.
