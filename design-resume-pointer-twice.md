# The orch resume pointer arrives twice after a compact cycle

`resume-pointer-twice` · Task · bug, orch · running · Drift-Baseline

## Seen

After a requested compact cycle of the orch (2026-10-31), the resume pointer reached the session twice: once as SessionStart hook context and once typed by the watcher.

- The hook runs `orchengine.ResumeContext` (`internal/orchengine/resumecontext.go`), which writes the resume mark with `At` from the hook's own clock.
- The watcher's skip check `hookDelivered` (`internal/orchengine/watcher.go`) counts the pointer delivered only when `mark.At` is not before `st.CompactionBaseline`.
- `tickCompacting` sets that baseline from the transcript's `compact_boundary` record, which Claude Code writes only after the SessionStart hook finished.
- Transcript times for the cycle: hook output 17:15:49.369–.384, `compact_boundary` 17:15:49.402, typed pointer 17:15:55.340.
  The mark is some 20–30 ms older than the baseline, so `hookDelivered` returns false and the watcher types the pointer.

## Idea

Compare the mark against a time taken before the hook can run, such as the moment the watcher entered the compacting phase or typed `/compact`, instead of the boundary record written after the hook; keep the text match.
A test covers a boundary stamped after the hook's mark.
