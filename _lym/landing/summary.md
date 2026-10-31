# Skip the typed resume pointer when the hook already delivered it

After a compaction of the orch session, the resume pointer reached the session twice: once as session-start hook context and once typed by the watcher.
The watcher's `hookDelivered` check compared the hook's resume mark with `State.CompactionBaseline`, which is set from the `compact_boundary` record.
Claude Code writes that record after the hook has run, so the mark was some tens of milliseconds older than the baseline and the check failed.

The check now compares the mark with a new persisted field, `State.PointerMarkSince`, in `internal/orchengine`.
A reload records in it the compaction baseline as it stood before the reload moved it to the new boundary.
A mark counts as the hook's delivery when its time is at or after that field and its text equals the pending pointer.
The text match is unchanged, and the hook verb, `ResumeContext` and the mark file's format are untouched.

## Design decisions

- Both compaction paths record the field: the requested compact cycle (`tickCompacting`) and the automatic compaction read at a turn end (`startAutoReload`).
  The previous baseline precedes this compaction's hook and follows every earlier compaction's hook, so a stale mark with the same text cannot pass.
- The reload after `/clear` (`tickClearing`) records the baseline unchanged, which keeps its current behavior.
- The field is persisted so a watcher restarted mid-reload reads the same bound.
  `toIdle` and `ResetForFreshLaunch` zero it.
- A state file from an older binary decodes the field as zero, and a zero field accepts any mark whose text matches.
  The cleared mark on return to idle and the text match still guard that case.
- The hook's clock and the transcript's clock are assumed to be the same host's wall clock, as the existing check already assumed.

The package doc in `doc.go` describes the new check.
Tests cover a mark stamped between the previous baseline and the new boundary after both an automatic compaction and a requested cycle, a mark older than the previous baseline or with other text still typing the pointer, and the field returning to zero after the reload.

## To check by hand

- Run a real requested compact cycle of the orch and confirm in the transcript that the pointer arrives once, through the hook, and the log shows `orch: pointer delivered by the session-start hook`.
