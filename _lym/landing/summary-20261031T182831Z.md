# Count the hook's resume mark from the baseline before the compaction

After a compaction the orch's resume pointer reached the session twice: once as session-start hook context and once typed by the watcher.
The hook writes its resume mark with its own clock while it runs, and Claude Code writes the `compact_boundary` record only after the hook finished.
The watcher's pointer step counted the mark as a delivery only when it was not older than `State.CompactionBaseline`, which the reload had already moved to that boundary,
so a mark some tens of milliseconds older than the boundary was rejected and the pointer typed.

## What changes

- `State` gains `PointerMarkSince`, persisted as `pointer_mark_since`: the earliest time a resume mark counts as the hook delivering the pending pointer.
- Both compaction reloads, the requested compact cycle in `tickCompacting` and the automatic one in `startAutoReload`, set it to the compaction baseline as it stood before they move the baseline to the new boundary.
  The reload after `/clear` sets it to the unchanged baseline, which keeps that path's behavior.
- `hookDelivered` compares the mark with `PointerMarkSince` instead of `CompactionBaseline`; the text match stays.
- Every return to idle and every fresh launch zero the field.
- The package doc's description of the pointer step names the new bound and why it holds.

## Design decisions

- The lower bound is the previous baseline, not the compacting phase's entry time.
  The automatic compaction has no phase entered before its hook runs, so only the previous baseline serves both paths.
  It still rejects a stale mark: the hook fires only after a compaction, every return to idle clears the mark, and an earlier compaction's hook ran before its own boundary, which is at or before the previous baseline.
- The bound is persisted in `State` rather than kept in memory, so a watcher restarted mid-reload reads the same bound when its pointer step runs again.
- A state file an older binary wrote decodes the field as zero, which accepts any mark holding the pending pointer's text; the clear at idle and the text match still guard that case.

## Tests

- `TestWatcher_HookDeliveredPointerIsSkippedAndAnythingElseIsTyped` now starts from a previous baseline and adds a row for a mark stamped 30 ms before the boundary, which is skipped;
  the row for a stale mark now dates it before the previous baseline, which still types the pointer.
  Every row asserts the bound is zero after the reload ends.
- `TestCompact_HookMarkStampedBeforeTheBoundaryIsNotTypedAgain` covers the requested compact cycle with a mark stamped before the boundary.
