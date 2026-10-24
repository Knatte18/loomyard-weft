# webster: Master writes stuck after reset --to batch-start instead of running recover-batch

`webster-reset-reopens-dead-batch` · Task · bug, webster

Imported from [issue #863](https://github.com/Knatte18/loomyard/issues/863).

Seen 2026-10-24 in run gate-runs-once: batch 22 ended dead (recovery_retry false). The way forward lyx prints (existingReportRemedy in internal/websterengine/beginbatch.go) is reset --to batch-start --batch 22, then lyx webster recover-batch 22. After the orch ran the reset and lyx loom resume, each fresh Master read status, saw batch 22 still dead/terminal with recovery_retry false (RecoveryRetry is always false once the reset cleared RecoveryStartSHA) and wrote outcome stuck again per webster-template-master.md; the tackler handed back twice.

Workaround: the orch ran lyx webster recover-batch 22 by hand from the task worktree, then lyx loom resume.

Expected: after reset --to batch-start, the batch reads as recoverable (status or recovery_retry reflects the cleared count) and Master runs recover-batch itself, or the reset re-opens the batch so begin-batch runs it again.