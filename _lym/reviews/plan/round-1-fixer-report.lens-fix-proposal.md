```yaml
lens: fix-proposal
outcome: reported
```

Fix-proposal lens: I tested each proposal the fixer carried out as proposed, looking for concrete cases where the proposed change is wrong.

1. **MEDIUM (F8)**: in card 1's test paragraph, `TeardownLockHeldPastTheWaitOwesTheRecordsPush` is listed as an owed row of the wrong test.
   - **Changed text:** "`TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide` is tier1 and runs each of its owed rows once more in the mode… `TeardownLockHeldPastTheWaitOwesTheRecordsPush`: the teardown_lock class".
   - **Summary:** that row belongs to another test, which card 1's **Edit:** list does not name, so the variant cannot be added as the card writes it.
   - **Evidence:** in `internal/fabricengine/pushanchored_integration_test.go`, `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide` runs from line 205 and `TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks` starts at line 646. The row is at line 796, inside the `TestPushLock_…` function. The review's premise that `TestPushPairAnchored_…` "already builds every owed setup" fails for this row.
   - The `Deferred_HeldWeftWriteLock` row (lines 399 and 417) and the `PrimeRecordsPushOwed_*` rows (lines 575 to 620) are in the named test.

2. **MEDIUM (F5 and F13)**: the new ended-set reads in card 2 apply to more branches than the proposals asked for.
   - **Changed text, `RunCommitsAhead`:** "A pair's records branch whose run the ended set reads ended reads level too, whatever its unpruned tracking ref holds".
   - **Changed text, `TipsOnOrigin`:** "A pair whose run the ended set … reads ended reads held on both sides, whatever its unpruned tracking refs hold".
   - **Summary:** both proposals limit the rule to a branch that origin no longer has. The fix drops that condition, so a branch still on origin, whose run ended on the board, also reads level and held while it carries unpushed commits.
   - **Concrete case:** a done run's pair has commits made after its entry ended. The dobby's `StartIfAhead` never restarts its pusher for them.
   - **Evidence:** `internal/fabricengine/pushclassified.go:672-678`. The push reads `view.ended` only for a gone branch (`if err == nil && view.gone { view.ended, err = goneEntryEnded(...) }`) and pushes a present branch even when its entry is ended. The local reads now judge differently from the push they stand in for.
   - Card 10's Push still runs before `Level`, so the teardown does not lose the commits. The dobby's restart does.

3. **LOW (F1)**: card 4's `PushRun` bullet says the `*TakeoversError` branch "serves every prime push, one that names a run included".
   - **Summary:** when the named run is itself the taken-over one, `PushRun` used to return the `ErrHandedOver` that `HandOver` returns. Under the new branch it returns the later push's result, nil, so the caller no longer learns its own run was handed over.
   - **Evidence:** `internal/pairteardown/handover.go:85-87` returns `HandOver(...)` on a `*TakeoverError`, and line 99's doc says it returns an error wrapping `ErrHandedOver` "so its caller stops". `HandOverOwnedElsewhere` (`handover.go:169-172`) and `catchup.go:277-280` read `ErrHandedOver`.
   - Card 4 does not say whether the branch still returns `ErrHandedOver` when the pending set includes the named run.

No findings on F2, F3, F4, F6, F7, F9, F10, F11, F12, F14 or F15. F6 has a nil-`Pair` path in the code (`handover.go`: `if req.Pair != nil`), so its rows can be built.

Tool calls:
- grep of `PushRun`'s hand-over return, battencli's `ErrHandedOver` readers and the test function starts in `pushanchored_integration_test.go`: my context lacked `PushRun`'s body and the function boundaries around lines 396 to 826.
- grep of the cancel and re-adopt `ErrHandedOver` handling, plus `pushclassified.go:660-680`: my context lacked where `view.ended` is computed. That grep found no `ErrHandedOver` handling in `cancelcmd.go` or `readopt.go`.
