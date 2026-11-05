---
round: 1
focus:
  - "internal/shuttleengine/rundir.go, wait.go, activity.go: does the run-end stamp get written on every finalize path (done, failed, cancelled, error), and is a stamp that cannot be written or read handled without losing the old fallback?"
  - "internal/shuttleengine/activity.go ReadNewestRunEnd: does a stale stamp from an earlier run in the same directory or slug ever hold a notice off for a run that is genuinely quiet?"
  - "internal/battencli/wire.go: does the live-wait count cover the record's three activity sources (gate-fixer sessions, a running `lym gate test` of the run, the row hand-over after a writer ends), and does it count only processes registered under this run's pair slug, so another run's gate cannot hold this run's notice?"
  - "internal/battencli/wire.go: can a dead or recycled pid, a detached runner that exited without cleanup, or an unreadable registration read as live and suppress a notice forever?"
  - "internal/dobbyengine/runnotice.go: does the one-line change keep the threshold semantics, so a notice still fires when none of the counted sources shows activity for the threshold?"
  - "Test coverage: do wire_seams_test.go, activity_test.go and wait_test.go exercise the two reported cases (gap after a writer's shuttle ends, idle writer with a running fixer), and the no-activity case where the notice must still fire?"
  - "Commit message and `internal/shuttleengine/doc.go` against the diff: does each claim (stamp at the run-directory root, cold fixer pane process, detached gate runner) hold in the code?"
---

The measuring stick is the board entry under `## Goal`: the record's other sections are empty.
The two bugs named there are the writer-ends gap and the idle writer with live fixers, so focus goes to whether both are closed and whether the fix can over-suppress notices.
