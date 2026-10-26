```yaml
lens: board-fit
outcome: reported
```

Task: board-fit review of the decision record against the board, its peers and parent commits.

**Findings**

1. `LOW`, `consistency`, Open risks (first bullet): this risk asks the parent whether to keep the removal of promote, but the entry has already decided that.
   Evidence: the entry body lists `lyx beam promote` under "Removed", "since a beam is made from its project, never from a running task".
   Note `beam-promote-keeps-discussion` already says "Until then … `lyx beam promote` is not used".
   Fix: drop the "keep the removal?" question.
   One part could stay as a writer-settled follow-up: notes `beam-promote-keeps-discussion` and `loom-beam-darns` describe turning a task worktree into a beam, which no mechanism will support after D13.
   That is not a parent decision, so it should not be raised as one.

2. `LOW`, `consistency`, D5 and Scope ("README's Running Owner column"): note `readme-per-project-files` plans to change README rendering per project, including what is root.
   The two only overlap as text in `render.go`, and the terms agree ("root project").
   The record could cite the note in its Out list so the README's layout is clearly not this task's.
   No conflict.

**Checked**

- `lyx loom peers`: no peers and no parent commits listed, so there are no run conflicts.
- Decision notes for this task: none in the `lyx board list` titles, so none to read or unreachable.
- `beam-promote-keeps-discussion` (E1): overlap, covered by finding 1; no conflict.
- `loom-beam-darns` (F20): overlap, covered by finding 1; no conflict with the entry.
- `loom-finalize-merges-itself` (E11): the overlap is text-only in `landingdeps.go` flags; that note hands pawl to this task. No conflict.
- `adopt-check-visible` (E12): D6 leaves step 6 unchanged. No conflict.
- `readme-per-project-files` (E6): text overlap, finding 2. No conflict.
- `darn-drift-await-row` (E8): none.
- `workspaces` (P1): none. It is a grouping project, and under D5 it reads root's owner.

**Tool calls**

- `lyx loom peers`: my context did not have the peer and parent-commit list.
- `lyx board get readme-per-project-files`: my context had only its title and brief, not the body that names the root project.
