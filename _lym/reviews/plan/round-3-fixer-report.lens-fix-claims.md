```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: checked every claim the fix added or rewrote in cards 3 and 8 against the code at HEAD.

no findings

Claims checked and found to hold:
- The F1 refusal claim, "refuses with an error wrapping `ErrPairPushRunning` ... as `HandOverTips` refuses": `internal/fabricengine/handover.go:71-77` takes the lock first and refuses with `ErrPairPushRunning` before any fetch or write.
  The only thing created earlier is the lock directory, through `ensureTeardownLockDir`.
- The claim that `archiveOwnerSides` is for a caller already holding the lock, "as `HandOverTips` uses it": confirmed at `internal/fabricengine/ownerarchive.go:87` and `handover.go:88`.
  `ArchiveOwnerTips` is the lock wait plus `archiveOwnerSides` and an `ErrArchiveFailed` wrap.
  Card 3 never promised that wrap for `TakeOverRun`, so moving the parent pair to `archiveOwnerSides` drops nothing the card stated.
- "No step takes a lock the call already holds": no lock acquisition shows in the first 40 lines of `DropLocal`, and none of `fabricengine`'s non-test lock sites is reached by step 3.
- The F3 claim that `hubFixDeps` passes "the subject's tries as a one-entry map": it matches card 2's `FixHold(boardDir, here, fixTask string, tries map[string]int)`.
- The new row `TheCheckOfASubjectOwnedElsewhereNamesTheRerun` says "no gate runs": `internal/adoptcli/check.go:158-164` posts the owned-elsewhere line and returns before the lock or the gate.
  `TheFixAtTheLimitNamesTheRerun` relies on the limit line carrying the re-run, which `fixloop.go:250` does.
- The F5 claims: `greenDeps.Dir`'s only read is the count reset (`green.go:36`).
  `greenFixture`'s `dir` and `otherDir` are read only by the "writes the try count to 0" subtest (`green_test.go:148-162`) and by the fixture's construction.
- The F2 claim that an aborted rebase leaves the branch at its local commits without the entry commit holds, because the card orders the drop before the rebase.
  On the conflict row the parent branch then ends at the planted local commit.

Tool calls:
- Read `HandOverTips` and `archiveOwnerSides`'s callers: context had only grep hits, not the refusal's order.
- Read card 2's `FixHold` signature, `runCheck`'s owner path and the green count-reset test: context had none of their bodies.
- Read `ensureTeardownLockDir`: to check whether "before anything is written" holds.
- Read `ArchiveOwnerTips`'s body: to check what moving to `archiveOwnerSides` drops.
