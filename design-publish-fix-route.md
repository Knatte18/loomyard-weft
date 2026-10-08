# A blocked Publish has no route that hands a finding to a fixer

`publish-fix-route` · Note · bug, loom, burler

Imported from [issue #462](https://github.com/Knatte18/loomyard/issues/462).

Seen 2026-10-08 in `shuttle-send`: Publish blocked three times on the same `publish_verify` failure (three `//go:build tmux` loomcli tests), and the two fix routes the hub has from a blocked Publish never reached the defect.

The defect arrived through the merge of main (reed-view typed `/color` raw into the pane), so it is outside the task's diff.

- `lyx loom decision add --by parent` plus `lyx loom goto --to Webster-Burler`: the Webster-Review rubric measures against the plan, not the decision record, and the reviewer reads only `git diff $(git merge-base main HEAD)..HEAD`, so both rounds approved with unrelated NITs and the decision was never acted on.
- The Webster-Burler verify gate compiles the `tmux` tier and never runs it (`internal/loomshed/doc.go`), so the fixer never saw the failing tests either.
- `lyx loom reject` works only at PR-Gate or PR-Rework, and `goto` refuses PR-Rework from Publish because it lies past the current row, so the hub cannot hand the failure to a fixer as a finding.

The hub fell back to `goto Plan-Bouncer`, so Plan-Review raises the uncarried decision against the plan, which re-runs Plan-Burler, Batchifier and Webster for one fix.

Expected: a blocked Publish (or any blocked row after Webster) has a route that hands a named finding to a fixer round, e.g. `lyx loom reject` accepted at a blocked Publish, or a Webster-Burler round that takes the publish_verify failure and post-Discussion parent decisions as its focus.
