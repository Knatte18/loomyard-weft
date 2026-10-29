```yaml
lens: code-tests
outcome: reported
```

Task: lens code-tests — for each behaviour the diff adds or changes, name the test row that pins it.

**Per-behaviour list**

- Refused forward of an unread chase still held by its holder: pinned by the changed row `TestScheduler_Chases/AChaseTheHolderStillHoldsUnreadIsRefusedForwardingAndStaysWithItsHolder` (`internal/creelengine/deliver_test.go:2616-2640`). Its `wantErr` asserts the refusal text, the way forward and `which answers #1`. `Forward` (`verbs.go:593-594`) is the production code under test, and the row is its only coverage of the chase branch.
- Refusal writes no journal entry: pinned by the same row's `before` and `after` count of `e.store.Log`.
- Chase stays unread with its holder: pinned by the same row's `Inbox` assert of `{From: creel, Unread: 1}`.
- Chase clock still reaches the first chase under the question sender's own hold: pinned by the same row, since `creelPostTo(e, creel, alpha)` returns a chase id only if the chase was posted. A broken exemption (`chase.go:133`) would give id 0, so the `wantErr` text would not match. The hold-clock row (`deliver_test.go:2426`, `ADormantHolderAndAnotherSendersHoldStopTheClockWhileTheMessagesOwnSendersHoldDoesNot`) also covers the own-sender exemption.
- Worker leaves the chase untyped under orch's hold (the determinism the diff adds): pinned by the same row's journal-count and inbox asserts, which would fail on a `read` entry. Nothing outside this row pins it, and it is test-infrastructure behaviour, not production behaviour. The hold-owner-only delivery filter (`deliver.go:238` and `deliver.go:545`) is covered by the existing `TTLRunsUnderAnotherSendersHoldAndExpiresUntyped` row (`deliver_test.go:808`).
- `Store.Hold` permits the orch strand as a privileged sender holding alpha: pinned by the row's `t.Fatalf("Hold: %v", err)` check, and by the hold-clock row at `deliver_test.go:2429`.

**Findings:** none.

- No uncovered behaviour.
- No duplicate test: the diff edits one existing row and adds none.
- No real-time sleep was added.
- `t.Parallel` and the injected clock are unchanged.

**Tool calls:** none made. The context already held the diff, the row, `Forward`, `ageChases`, `snapshot` and `Hold`, and the 20-run gate result.
