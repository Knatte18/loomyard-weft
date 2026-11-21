```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I ran each changed function and branch on a concrete case.

**Per function or branch: the case I ran, and whether it is correct**

- `redFirstGate.arrival`, entry without the bug label: passes, `heldTree` cleared. Correct.
- `arrival`, "No test seam" record lands while held: `exempt` is read before the tree check, so it passes and the hold lifts. Correct.
- `arrival`, first exit 4 or 5: answers pending, sends the hold line, sets `heldTree` to `res.Tree`. Correct.
- `arrival`, held and HEAD tree unchanged (worktree clean or dirty): answers pending, runs nothing, sends nothing. Correct.
- `arrival`, held and a new commit:
  - check exits 4 again: verify runs once more and the answer is pending with no line. Correct.
  - check exits 0: passes, hold cleared. Correct.
  - check exits green: an ordinary failure, `heldTree` cleared, one attempt counted. Correct.
  - check exits on timeout or runner crash: `holdsForCommitOrRecord` is false, so the result is a terminal failure. Correct.
- `arrival`, `HeadTree` fails while held: wrapped and returned, the run ends on a gate error. This matches how every other read error is handled. Correct.
- `arrival`, verify errors while held: returned, `heldTree` left as it was; the run ends anyway. Correct.
- `arrival`, the hold line's send fails at the wait loop: `heldTree` is already set, so every later re-check answers pending with an empty `Send`, and the line is never retried. **Defect: F1.**
- `redFirstGate.final`, deadline while held on an unchanged tree: runs verify again on a tree the hold already judged, then fails with the old findings. The verdict is right; it costs one extra red-first run at the deadline. **F3, LOW.**
- `final`, entry exempt: passes. Correct.
- `exempt`, labels error or record read/parse error: a wrapped error. Correct.
- `exempt`, called every 5 s while held: reads the record each time, so a record written mid-hold lifts it. Correct.
- `verify`, StatusDirty while not held: gives the dirty verdict. Correct.
- `verify`, StatusSkipped: only possible from a recorded pass on the same tree, so it passes. Correct.
- `holdsForCommitOrRecord`, `ExitCode -1` (shell could not start): false, so terminal. Correct.
- `redFirstHoldLine`: both texts are single lines; the 4 and 5 branches are distinct. Correct.
- `resolveGateEntry` and `resolveRedFirstGate`, red-first with a missing seam: the error is returned. Correct.
- `resolveGateEntry`, the other entries: `final` is nil, so `MayHold` is false and they keep their previous contract. Correct.
- `resolveGateEntry`, verify entry with `VerifyReach`: unaffected by the change. Correct.
- `preflightTableError`, a wrapped `*NewerBoardCopyError`: `stencilstore` returns a pointer (`skew.go:89`), so `errors.As` matches and only one way forward is left (`skew.go:39`). Correct.
- `preflightTableError`, any other error: gains the config clause. Correct.
- `latestDarnOutcome`, latest entry Done while an earlier file is kept: names no file. Correct.
- `latestDarnOutcome`, latest entry is a Stuck that wrote no findings file while an earlier halt's file is kept: names the stale file. **Defect: F2.**
- Removal of `clearGateFindings` in `burlerfix`, `burlerreview` and `multillm`: no other state depended on it. Correct.
- `HeadTree` export: a rename only. Correct.

**Findings**

**F1 — MEDIUM, design.** `internal/loomshed/redfirstgate.go`, `arrival`'s hold branch (diff 005, lines 160–166).
- The gate sets `heldTree` before it learns whether the hold line was delivered.
- When the send fails, `handleGatedBoundary` (`internal/shuttleengine/wait.go` ~577–580) logs and returns, leaving the writer at the boundary.
- Every later re-check on the same tree answers pending with `Send == ""` (diff 005, lines 142–143). The line is never retried.
- The writer is never told to ask the parent for the record, so the run can hold silently until its deadline. `redFirstHoldSendFailed` only logs this.
- The wait loop's contract ("a failed pending send … leaves the entry pending") expects the closure to send again.
- Fix: keep sending the line until it is delivered. For example, resend on every pending answer until the writer's next turn end, or track delivery separately from `heldTree`.

**F2 — MEDIUM, design.** `internal/loomcli/darnwiring.go`, `latestDarnOutcome` (diff 004, lines 28–33), together with the removal of `clearGateFindings`.
- Sequence: a halt keeps `Darn-Write.md`; the run resumes; Darn-Write passes, and the file is now kept. Later Darn-Write is called again (goto) and ends Stuck by a path that writes no file:
  - a scope refusal (`darnwrite.go:144`);
  - a stop report (`darnwrite.go:182`, `:185`);
  - a session Stuck (`darnwrite.go:188`);
  - a terminal gate reason, which writes no file (`gateattempts.go`, the `gate.Reason` return).
- In each case the latest Stuck still names the stale file, and `renderPriorWork` (`darnwrite.go:293–294`) tells the writer to "read them and fix what they name".
- Before the diff, the intervening pass had removed the file.
- Fix: only name the file when it was written by the halt the latest Stuck records. For example, compare the file's mtime with the entry's time, or record the findings path on the history entry.

**F3 — LOW, design.** `internal/loomshed/redfirstgate.go`, `final` (diff 005, lines 170–187).
- At a deadline or liveness finalize while held on an unchanged tree, `final` runs `verifytree.Verify` again. Verify records passes only, so the full red-first check runs a second time on a tree whose refusal is already known.
- This is the focus's "runs the check twice on one tree".
- Fix: when `heldTree == HeadTree()`, build the failure from the held result instead of running the check again. That means keeping the last `verifytree.Result`.

Tool calls:
1. `grep` of `internal/stencilstore` for how `NewerBoardCopyError` is returned and its `Error()` text. My context did not show whether it is returned as a pointer, which `errors.As` with `*T` needs.
