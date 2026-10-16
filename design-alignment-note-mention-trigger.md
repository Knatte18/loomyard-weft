# A note that only names a running task stops the alignment agent from settling its PR

`alignment-note-mention-trigger` · Note · bug, loom

## Seen

On 2026-10-16 the PR-Gate alignment agent of `creel-stage1-fixes` did not settle PR #691: "a decision was recorded for this task while the alignment agent ran".
No decision was recorded; the decision record was unchanged.
The trigger was the new note `slug-reuse-old-pr` (#692), whose body names `creel-stage1-fixes`, which changed the snapshot of notes naming the slug.
The orch then had to review the PR with a fork.

## Idea

The alignment agent counts only a real decision on the task (the run decision record, or a board note recorded as a decision for that held task) as a change, and not any note that only mentions the slug.