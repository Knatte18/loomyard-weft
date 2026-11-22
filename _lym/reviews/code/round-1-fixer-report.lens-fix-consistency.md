Task: check the fix-consistency lens over the change set. There are three LOW findings and nothing at higher severity.

1. **LOW** — `TestNewCommitStatusSeam` doc comment, `internal/battencli/commitstatus_test.go:32-33`.
   - The comment says "an uninterrupted start commits each row from Seed-Child to Run-Shed without a push, while Worktree-Create, a resumed or halted start row and every later row push."
   - The fix added two rows. One is the start-row marker repeat (`StartRowWritesTheMarkerSoARepeatSkips`). The other is `WorktreeCreateAfterAStartRowPushes`.
   - The fix also added a gate-admission assertion (one admission per push).
   - None of these appear in the comment, which lists the covered behaviours. It is stale for those two rows and the assertion.

2. **LOW** — seam doc in `internal/battencli/commitstatus.go:~219-221` (the "after-commit callback rewrites the marker right after a successful Commit and before the push" paragraph).
   - The callback now writes `push_pending` as true when a commit was made. The seam then rewrites the marker with `push_pending` false once the core returns nil.
   - "Rewrites the marker" is still true, but it omits the second write, so a reader misses the pending lifecycle. The `commitStatusMarker` doc does describe it.
   - The text two lines above now says a start row after a failed push pushes. That paragraph agrees with the new behaviour, so this one is the twin left unchanged.

3. **LOW** — `pattern/PATTERN-run-fsm.md:43`, with the same sentence in the `PATTERN-run-fsm` entry in `PATTERN.md`.
   - The body says "a resumed or halted start row pushes as any transition does."
   - The F2 fix means a start row after a failed or interrupted push also pushes. The PATTERN text was not touched for this.
   - The PATTERN text now understates the exception, though it does not contradict the code.

Checked and consistent:
- `commitStatusMarker` has two other users, `lifecycle_integration_test.go:1435` and `shedrun.LastCommitMarker`. The `omitempty` JSON tag keeps both compatible.
- The only other `CommitStatus` wirings are the loom seam and tests, and the fix leaves them untouched.
- `c.commitStatus` is set before `shedPaths` is built in `wire.go`, so `CommitStatus: c.commitStatus` is never nil there.

Tool calls (2):
- `grep` for `commitStatusMarker`, `LastCommitMarker` and `CommitStatus:` across the repo, plus `grep` for push and Seed-Child in `PATTERN-run-fsm.md`. My context lacked the other marker users and the PATTERN body text.
- `sed -n 1415,1460p` of `internal/battencli/lifecycle_integration_test.go`, to see how that test writes the marker. My context lacked that file.
