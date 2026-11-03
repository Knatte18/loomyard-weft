```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked the ways forward the fix added in D2, D3, D7 and D8 of `_lym/discussion/decision-record.md`.

**MEDIUM**: D2's Land restore after a removal leads to a fresh warp branch, which contradicts D1.
- Changed text: D2, Land, not-live case: "`git -C <records repo> branch <weft branch> <tag>` … the re-run's D1 adopts it". D1 step 3: "Under the adopt-only option `Add` forks nothing".
- Problem: at Land, the removal that archived the weft tip has usually also deleted the task branch, both locally and on origin, because its work landed (`deleteWarpBranch` and `deleteTaskBranchOnOrigin` in `internal/fabricengine/remove.go`). Restoring only the weft branch makes the pair live. `Add` then finds no local warp branch to adopt, and `probeWarpLeftover` returns no origin tip. So `Add` forks a new warp branch from the parent's head or from `HEAD` (`add.go:419-430`).
- Consequence: the adopt-only `Add` does fork, against D1's claim. Nothing shows that Land can resume on a task branch that holds none of the run's commits. The way forward either has to restore the task branch too, or D1 and D2 have to say what Land does over a fresh warp branch. The D2 Land acceptance criterion does not cover this.

**MEDIUM**: The Finalize adopter-first stop leaves the batten record with no step that clears it.
- Changed text: D3: "The record stays under the owner pause, which keeps a pass from driving it, and is left to the adopt that clears that pause." Also: "a slug an owner pause covers, get no warning."
- Problem: the stop now names no cancel, and `lym fabric remove` gives no warning while the owner pause stands. No message names a command for this machine's leftover batten record.
- Evidence: the only adopt the halt texts name is `lym adopt <subject>` (`ownerHaltReason`, `finalize_owner.go:100-124`). That adopt takes the run over and resumes it. Here another machine has already landed the run, so taking it over is not clearing a finished record. The text "the adopt that clears that pause" names no command, and the record is left with no way forward.

**LOW**: D2's restore from "any clone" names no command it can fill in.
- Changed text: D2: "With no archive tag on origin … the stop names the same `git branch` restore from any clone that still holds the branch".
- Problem: the pre-run cannot fill in a source for the `git branch`, so this way forward is only a direction. It should either name a command, or name the reflog of the local records repository, which the pre-run can read.

I checked these and found nothing wrong:
- The `ErrRemoteLeftover` commands: once they run, `probeWeftLeftover` reads the local branch with nothing on origin as live.
- The re-run over a `failed` status: `checkRunStatus` resumes it.
- D7's refusal of a hand `Add` or a cancel's discard: it names the re-run.
- D2's step "`lym fabric remove <slug>` for a records worktree that still stands": `lym fabric remove` skips its reconcile when the task worktree is gone (`internal/fabriccli/remove.go:43`), so the step can run.

Tool calls:
- Read `internal/fabriccli/remove.go:180-235` to see whether `lym fabric remove` hits the run-scope reconcile error on a gone task worktree; my context had only its call-site line.
- Grepped `internal/fabriccli/remove.go` for its gone-task-worktree handling; my context did not have that file's flow.
