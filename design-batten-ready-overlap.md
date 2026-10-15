# Start one task per overlapping group at a newly-ready start

`batten-ready-overlap` · Note · enhancement, batten

Split from `sibling-overlap-check` on 2026-10-15; the operator decided only the board warning and skill rule there.

## Idea

When a landing makes several tasks ready at once, batten starts one task per group of overlapping tasks (shared module labels, later the paths of note `labels-from-diff`) and leaves the rest for the orch, naming the overlap in the notice.