# Card 17 — boardcli-wiring

**Edit:**
- `internal/boardcli#Command`
- `internal/boardcli#storeVerbs`
- `internal/boardcli#outputGetTask` -> `func outputGetTask(out io.Writer, task *boardengine.Task, parentBeam string) int`
- `internal/boardcli_test#TestCLI`

**Create:**
- `internal/boardcli_test#stepProjects`

**Extends:**
- `internal/boardcli/doc.go`

**Uses:**
- `internal/fabricengine#ValidatePairSlug`
- `internal/hubgeom#ParentOf`
- `internal/boardengine#Board.ParentBeamOf`

**Intent:**
`boardcli`'s pre-run fills the board's new seams from the resolved location:
`Config.ValidateSlug` calls `fabricengine.ValidatePairSlug` with the hub's configured junction names (`fabricengine.LoadConfig(ConfigDirs(layout)).Dirs()`), so a new task or project slug, a promote and a rename obey fabric's rule (D14);
`Config.PairExists` reports whether the slug's pair worktree exists under the hub (`fabricengine.WorktreePath` of the prime's location), for D2's project-change and rename refusals;
`Config.ParentOf` is card 15's `hubgeom.ParentOf` over the prime's location.
The `--board-path` path sets none of them, as today.
`lyx board get` carries the entry's `parent_beam` beside `task` in its envelope (`outputGetTask`), from `Board.ParentBeamOf`, and gives a project its effective owner as it gives a task's; `lyx board list` already carries it from card 12.
The `board` command's `Long` names the project kind, its fields and `parent_beam`.
`stepProjects` runs from a seeded hub: an upsert of a project and a member task, `list` and `get` showing `parent_beam` (empty for a grouping project, the beam once the test hand-marks the project `beam: true` in `board.json`), an upsert of a task slug ending in the records suffix, and one equal to a configured junction name, each refused with fabric's reason, and a member's `project` change refused naming `lyx batten cancel <slug> --remove` while a directory stands at its pair worktree path.
`TestCLI` runs it as one more step.

**ImpactSummary:** The board CLI wires three seams and adds one get-envelope key; outputGetTask's two callers are in storeVerbs on this card.
