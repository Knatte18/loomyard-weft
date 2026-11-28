```yaml
lens: fix-claims
outcome: reported
```

Fix-claims lens: I checked whether each sentence the fix added or rewrote holds against the code at HEAD.

no findings

Claims that hold:
- `hubforge.AddPair` pushes both sides: it calls `AddPairWith`, which runs `CoalescePushBothAt` (hub.go:485-499).
- `lymcwd.ResolveWorktree` exists (lymcwd.go:93).
- `boardengine_test` can import `internal/pairteardown` without a cycle. `pairteardown` imports `boardengine` (originseams.go), and an external test package may import it.
- `pairteardown.New(prime *lymcwd.Location)`, `OriginSeams(SeamOptions)`, `HubSeamOptions(prime)` and `OwnerReader(identity, parentOf)` match the signatures the card uses.
- Leaving `LivePair` nil is safe: the moves code checks it before calling it (originreconcilemoves.go:528).
- The `OwnerOf`-error row is reachable with board bytes that do not decode. `ReconcileOrigin` calls `readOriginOwners` (originreconcile.go:571) before `readOriginEnded` (:575), and `OwnerReader` wraps `ParseOwners`' decode error.
- `originCommit` sits in `replay_integration_test.go` and `originCommitFile` in `originreconcilemoves_integration_test.go`.
- None of the files holding the moved tests mentions heddle.
- The empty-subject fallback is in `fabriclayout` (origintypes.go:195), the outcome type `ClassifyEntries` returns. That fits card 9's claim that `TestClassifyEntries` holds the rule.
- Card 10's `TestEndedTeardowns` is the core acceptance row of the teardown report (card 10 lines 58-65).
- The teardown-in-tier1 question is still marked Unverified. `teardownEnded` runs `t.Run`, so whether that needs tmux is still open.

Outside my lens: card 10 line 65 lists the earlier-snapshot-tag and run-scope rows as deleted in card 12, but card 12 keeps both. That text was there before this fix.

Tool calls:
1. Read `readOriginOwners` (originreconcile.go:718-760) and grepped the empty-subject check, to see where the error row's bytes come from.
2. Grepped card 10 for `TestEndedTeardowns` and read `teardownEnded`, to check the report-copying claim and what the teardown runs.
3. Grepped the order of the owner and ended-set reads in `ReconcileOrigin`, to check the error row reaches the owner reader first.
4. Grepped `fabricengine` and `fabriclayout` for the empty adopt-subject fallback, to check card 9's claim.
