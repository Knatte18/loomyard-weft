# Intake closes an issue the main branch already fixed

Board intake now checks whether an issue's fault is already fixed before it puts the issue on the board.
Imported bugs whose fault was fixed on the base took run slots in a full queue and found nothing to do.

## What changes

- `contracts/orchrole/role-intake.md` gives the intake fork a fourth outcome beside a new note, a fold and a close as noise: a close as already fixed.
  The fork judges it from the main branch's code and history, read with git and never checked out or changed.
  It looks at the code path the issue names and the commits that touch that path since the issue was filed or since the build it reports.
  It closes the issue with `"completed":true`, its reason naming the fixing commit on the main branch and what the fork saw on the path.
  When the fork cannot tell, or cannot name the fixing commit, the issue goes on the board as before.
- `lym board intake close` keeps its payload.
  Its help and the `ly:board` skill now show the already-fixed form, a reason naming the commit with `"completed":true`.

## Design decisions

- **No new `close` field for the fixing commit.**
  The free-text `reason` already carries the commit to the issue's closing comment, and `completed` already closes it as completed.
  A structured field would need the CLI to resolve the commit against the main branch, a git read that only fabric may do,
  for a value only a human reads on GitHub.
- **No fixing commit, no close.**
  A fault the fork believes gone but cannot tie to a commit goes on the board, so a darn run settles it as before.

## Merge danger

**Door:** two-way
**Blast radius:** intake
