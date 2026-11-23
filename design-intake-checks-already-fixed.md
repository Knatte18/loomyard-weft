# Intake checks whether an issue is already fixed before it reaches the board

`intake-checks-already-fixed` · Task · enhancement, board · running · Describe

## Seen

On 2026-11-23 three imported bugs ran as darn tasks and each found the fault already fixed on the base:
`pr-findings-open-fabric-nil` (fixed by `6fa7f1740`), `amend-plan-loses-webster-batches` (no losing path left) and `rules-read-plan-gated-403` (fixed by `895eab670` two days before).
Each took a run slot in a full queue for nothing.

## Wanted

Operator decision (2026-11-23): board intake checks whether a bug is already fixed before it puts it on the board.

- The intake fork's rules, `contracts/orchrole/role-intake.md`, gain a fourth outcome beside new note, fold and close as noise:
  close as already fixed, its reason naming the commit on `main` that fixed it and how the fork saw it.
- The fork judges it from the code on `main` and its history, as the three runs above did in their first minutes:
  the code path the issue names, and commits since the issue's build that touch it.
- When it cannot tell, the issue goes on the board as now.
- `lym board intake close` may need a reason form that names the fixing commit; decide in the run.
