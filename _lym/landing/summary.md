# Shipped rule and docs name arras for a tapestry-sized note

The board-policy skill proposed a `motif` task for a note weighed for a tapestry run.
It now proposes an `arras` task, and keeps `motif` alone for a design decided now and split later on code still changing.

- `plugins/ly/skills/board-policy/SKILL.md`: the rule names `arras`, says a project's release is one release of its draft members, and gives `motif` its own bullet.
- `plugins/ly/skills/board/SKILL.md`: `arras` joins the recipe names and the arras run is described: discussion, a reviewed split, and a family filing that turns the entry into the project.
- `contracts/orchrole/orch-role.md`: the orch role lists `arras` among the inner runs and names it for a tapestry-sized note.
  It tells the orch that an operator decision on a running arras entry goes through `lym loom decision add --by operator`, never the entry body.
- `docs/reference/weaving-glossary.md`: `arras` is listed as the recipe for a tapestry-sized note; `cartoon` stays reserved.
- `internal/boardcli/cli.go`: the `recipe` help line of `lym board upsert` names `arras`.

## Design decisions

- The edits build on main's text of the board-policy skill after c8e43ac4, which this branch already holds.
- The release wording follows the decision record's auto-mode assumption: a project release releases its draft members at once.
