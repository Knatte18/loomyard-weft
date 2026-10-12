# Batten sends no notice for a landing that readies nothing

`batten-land-notice` · Note · bug, batten

Batten sends a notice at a landing only when the landing makes another task ready (`landed; newly ready: ...`, `internal/battenshed/land.go`, once per batten run).
A plain landing sends nothing, so the orch does not learn of it: on 2026-10-12 `intake-bugs-1012` (19:09) and `cleanup-1012` (19:19) landed through the alignment agent's PR approve with no notice, and the orch reported them as still running until the operator pointed it out.
With the alignment agent landing tasks without an orch turn, the landing notice is the orch's only signal to check the Land row, prod and the board intake.

## What to fix

Every landing sends `<slug>: landed` (with `newly ready: ...` appended when it applies), and a failed Land row still sends its failure notice as today.
