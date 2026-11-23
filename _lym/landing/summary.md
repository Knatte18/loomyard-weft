# Board intake closes issues the main branch already fixed

Three imported bugs recently ran as full tasks and each found its fault already fixed on the base.
Each took a run slot for nothing.
Board intake now checks for that before it puts a bug on the board.

The intake fork's rules in `contracts/orchrole/role-intake.md` gain a fourth outcome beside new note, fold and close as noise: close as already fixed.
The fork judges it from the code on the hub's main branch and its history, read with git and never checked out or changed.
It looks at the code path the issue names, and at the commits touching that path since the issue was filed or since the build it reports.
It closes the issue as completed, its reason naming the fixing commit on the main branch and what the fork saw on the path.
When the fork cannot tell, or cannot name the fixing commit, the issue becomes a note or a fold as before.

## Design decisions

- `lym board intake close` needs no new field.
  It already takes `"completed":true`, and the reason is free text, so the fixing commit goes in the reason.
  Only the command's help text and examples in `internal/boardcli/intake.go` change, and the `ly:board` skill documents the same form.
- A commit that cannot be named is not enough to close.
  This keeps the outcome conservative: a doubtful issue still reaches the board.

## To check by hand

- Nothing was run against a live hub: the new outcome is a rule for the intake fork, not code.
  Run intake once on an issue whose fix is on main and confirm the fork closes it as completed with the commit named.

## Merge danger

**Door:** two-way
**Blast radius:** board
