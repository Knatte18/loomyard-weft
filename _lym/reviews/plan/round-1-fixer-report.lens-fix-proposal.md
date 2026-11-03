```yaml
lens: fix-proposal
outcome: reported
```

Fix-proposal lens: I challenged the review proposals the fixer carried out as written (F1–F9, F11–F13, and F10's sentinel option). Three findings.

**F6 — MEDIUM.** Changed text: card 13 Wiring, the `battenCLI.preflightRunOwner` bullet.
- **Summary:** the follow-up goes on the end of the refusal, after its re-run step. The Run-Shed refusal then says to re-run the resume before the cancel, which is the wrong order.
- **Evidence:** `stoppedRefusal` (`internal/battencli/resumereconcile.go:156`) builds `"...; %s; %s"` from `branch.WayForward` followed by `reentry`. `resumeRefusal` passes `then re-run lym batten run &lt;slug&gt;` as the reentry (`resumereconcile.go:127`).
  - Appending `BattenFollowUp` to the end gives: "…run `lym fabric remove X` …; then re-run lym batten run X; then run `lym batten cancel X --remove` …".
  - If the operator follows that order, the resume after the removal reaches card 10's re-adoption. The weft is archived, so card 12 fails the run with a stop before the cancel ever runs.
  - The follow-up has to sit between the way forward and the reentry, or the reentry has to be dropped when a follow-up is added. As written, appending to the finished error cannot do either.

**F3 — MEDIUM.** Changed text: card 4 Intent, the reused-records-worktree bullet ("advances the branch with `git merge --ff-only &lt;origin tip&gt;` run inside the reused worktree").
- **Summary:** a reused records worktree with uncommitted run records in files the origin tip also changed makes `merge --ff-only` fail with "local changes would be overwritten". That error is outside D2's closed set, so card 12 reads it as transient on every pass, which is the failure F3 set out to end.
- **Evidence:**
  - The worktree is reused because a run died with its task worktree gone, so its records sibling can hold uncommitted records.
  - `merge --ff-only` refuses when the working tree has changes in a path the fast-forward rewrites.
  - The card gives no typed answer for this case, and no row covers it.
  - A dirty reused worktree needs a typed refusal in D2's set, or that branch must stay unchanged, with the refusal saying why.

**F9 — LOW.** Changed text: card 10, the sentence that `presentTaskWorktreePath`'s refusal "names `lym batten run &lt;slug&gt;` … in place of today's hand restore and run-directory deletion".
- **Summary:** the refusal is also reached from `lym batten cancel` and from batten's goto. There, naming only the resume drops the one non-resume way out the text used to give, so an operator who is cancelling is told only to resume.
- **Evidence:**
  - `childRestepped` and `childParentHolds` in `internal/battencli/cancelcmd.go:389,404` call `taskWorktreeLocation`.
  - `battenPreGoto` in `internal/battencli/goto.go:19` calls it too.
  - The card's sentence assumes the refusal is reached only "when the worktree disappears mid-run" during a pass.

Proposals I checked and found sound:
- **F8:** `remover.run` returns `pairteardown.Result` (`internal/fabriccli/remove.go:47`), so `res.Warning` is in reach, and `removeFields` has one production caller.
- **F1, F2, F4, F5, F7, F10, F11, F12, F13:** no case found where the proposed change is wrong.

Tool calls:
1. grep of `removeFields` callers and `fabric.go` 1105–1140, plus `remover.run`'s signature: to confirm `res` is a `pairteardown.Result` for F8. The context held only the call site.
2. grep of every caller of `taskWorktreeLocation` and `presentTaskWorktreePath` in battencli: to find every place the F9 refusal is reached. The context held only `wire.go`.
3. `cancelcmd.go` 375–412 and `goto.go` 10–25: to see the cancel and goto callers of that refusal, which the context lacked.
