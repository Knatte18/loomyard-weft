---
verdict: APPROVED
---

No findings. The diff makes the three changes the amended entry asks for, and each round-2 focus item settles as not a defect.

## Rejected

- Partial capture, then a replacement to off, then one to full (`RecordScope`): not a defect. The off replacement leaves the capture record alone, and the later full replacement compares the new skip list against `CaptureRecord.Skip`, which `RunCapture` records, so `dropCaptureSkipping` removes the capture. The chain subtest in `TestRecordScope` pins this.
- A failed removal of the capture record (`RecordScope`): not a defect. The removal runs inside the `UpdateScope` callback before the scope is assigned, so an error returns without a write and the old scope stays. If the removal succeeds and only the scope write fails, the retry finds no capture to drop and still writes the scope.
- Which `StartCapture` errors become the capture halt (`captureAgain`): only `childNoRecordError`, raised by `startFailure` when the child ended or stalled before its record, offers `lym drift defer`. A failed binary lookup, an uncreatable child log and a held capture lock stay errors, and the baseline test table covers each. `StartCapture` returns the `startFailure` error unwrapped, so `errors.As` matches it.
- A subdirectory anchor against the worktree root: not a defect. Every drift call site (`driftDeps`, `captureSpawnRequest`, `startMainCapture`, `runCaptureChild`, `runSideCapture` and the compare command) passes `location.AnchorPath()` as `AnchorDir`. The capture-job name still uses the worktree root, matching `pairteardown`. `TestDriftDeps_RunInTheAnchoredDirectory`, `TestCaptureSpawnRequest_StartsInTheAnchoredDirectory` and the driftcli anchor integration test would fail on a root-based call.
- `acquireSlot` against `Holder.Worktree`: the holder now records the anchored directory, and the field's doc comment says "its worktree root, or the anchored directory inside it". The holder's consumers only print the value.
