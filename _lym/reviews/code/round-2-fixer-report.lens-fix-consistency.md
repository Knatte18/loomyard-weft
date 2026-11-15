```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F2 and F3 change set for sibling or twin text left stale (lens fix-consistency).

**Findings: no findings.**

- The `ownerTick` doc comment in `internal/orchengine/owner.go` now agrees with the code. The exit reason is saved whenever `Stop` succeeded. A failed end returns `done=false` with the joined error. A failed stop returns before the save.
- The `internal/orchengine/doc.go` paragraph and the `FailingEndStillStopsTheStrandAndReturnsTheError` row both say the same. The row now expects `wantExit: "owned by machine there"` with `wantDone` false and `wantErr` true.
- The twin rows `FailingEndAndStopReturnTheErrorAndKeepTheWatcherRunning` and `FailingStopReturnsTheErrorAndKeepsTheWatcherRunning` still expect no exit. That is correct, because `Stop` failed in both.
- `internal/orchcli/doc.go:120`, "removes the strand even when that end failed", is still true and does not mention the exit reason, so it needs no change.
- The new `# Ending a subject owned elsewhere` section in `internal/pairteardown/doc.go` matches `internal/pairteardown/endsubject.go`. It sits before `# Quiet rule`, and the other sections that name the orch (the owned-process end and the bare session end) are unaffected.
- I found nothing else in non-test code or `pattern/` that says a failed end leaves the exit reason unrecorded.

**Out of scope:** F1 (`_lym/landing/summary.md`) was not touched, as expected. The review says it stands until the parent edits that file.

Tool calls:
1. `grep` over `internal` and `pattern` for the exit-reason and failed-end wording, to find twin text the fixer might have missed.
2. `Read` of `internal/orchcli/doc.go` lines 112–125, because the context did not hold that file.
