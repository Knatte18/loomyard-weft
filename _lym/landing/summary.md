# Print the prime as the root beam `root` on the board

`lym board list` and `lym board get` now print `parent_beam` as `root` for every entry whose nearest beam is the prime.
Before, the value was empty, which reads as "no beam", while the board view and window titles show the prime as `P0`.
`root` is `boardengine.RootSubject`, the slug that `lym adopt root` and `lym beam stop root` already take.

`lym board list --text` also shows the root project as a `P0` row with slug `root` and the title the board files give it.
The row sits before the first project, note or done entry, and last under `--sort`, since it has no dates.
It is left out under `--label`, because the root carries no label.
The JSON `tasks` array gets no root row.

## Design decisions

- The mapping is at the printing boundary only: `ListTasksBrief` in `internal/boardengine/store.go` and `outputGetTask` in `internal/boardcli/cli.go`.
  `boardengine.ParentBeam`, `Board.ParentBeamOf` and `hubgeom.ParentOf` still return the empty slug for the prime,
  since every owner read and beam verb takes that value as the prime.
  A parent that cannot be resolved still lists with no `parent_beam`, as before.
- The `P0` row comes from the new `boardengine.RootProjectBrief`, and `BriefTask.IsDone` lets `boardcli` place it ahead of done entries.
- The prose readers of the printed value move with it:
  `contracts/orchrole/orch-role.md` reads `root` as the prime and addresses its orch as `<shortname>:orch`,
  and `plugins/ly/skills/board/SKILL.md`, the `boardcli` help and its package doc say the same.

## To check by hand

- `p0-label-branch-untested` (design, medium) in `internal/boardcli/cli.go`:
  the `--label` branch that leaves the `P0` row out.
  `stepListAndFindText` in `internal/boardcli/verbs_test.go` now lists with `--label bug`; confirm it covers the branch.
