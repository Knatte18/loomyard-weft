# PR-Gate takes only its own run: no adopted old PR, no note-mention trigger

`pr-gate-run-identity` · Task · bug, loom, batten, board · running · Describe

Two PR-Gate faults where a run takes something that is not its own as its own. Both fixes sit in `internal/landingshed`, and the second also in `internal/loomcli`.

## 1. A reused slug adopts the old run's closed PR (issue #692)

Seen (2026-10-16): the board task `creel-stage1-fixes` reused the slug of an earlier task that landed the same morning (commit `27887027b`, PR #672 closed by Finalize at 06:47Z).
Two things then broke.
1) `lyx batten run` refused: the old run's record in `_lyx/shed/<slug>` still read completed; `lyx batten cleanup --apply` cleared it.
2) At Publish, `FindPullRequest` (`internal/landingshed/pullrequest.go`) lists PRs by head branch and base with state all and takes the newest, which was the old run's closed PR #672, so Publish halted with "the pull request was closed without being merged" and the tackler handed the stop to the orch.
Workaround: the orch opened a replacement PR (#691) from the task branch by hand and resumed the run.

Expected: a new run never adopts a PR it did not create, for example Publish records the PR number it created in the run state and ignores older PRs for the branch, or the board or batten refuses a slug whose earlier run landed.

## 2. A note that only names a running task stops the alignment agent from settling its PR

Seen (2026-10-16): the PR-Gate alignment agent of `creel-stage1-fixes` did not settle PR #691: "a decision was recorded for this task while the alignment agent ran".
No decision was recorded; the decision record was unchanged.
The trigger was the new note `slug-reuse-old-pr`, whose body names `creel-stage1-fixes`: `alignmentSnapshot` (`internal/loomcli/alignmentwiring.go`) hashes the text of every note that contains the slug, so the snapshot moved.
The orch then had to review the PR with a fork.

Expected: the alignment agent counts only a real decision on the task as a change, and never a note that only mentions the slug.
How a board note is recognised as a decision for a held task is a design call for the Discussion.
Orch recommendation, not a decision: the run decision record, plus a note whose title starts with a fixed form such as `Decision for <slug>:`, the form the orch already uses when the decision record does not exist yet; the orch role's decision-note rule would then name that form.

## Also

`decisionRecordedReason` is declared twice, in `internal/landingshed/pralign.go` and `internal/loomcli/alignmentwiring.go`, with the same text; keep one.
