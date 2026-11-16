# Board README entry cell shows pull_request and on_land

The board README's entry cell rendered a task's brief and one code-span bullet per `run_settings` leaf, and nothing else a run reads.
A task with `pull_request` on therefore looked like one with no pull-request override.

`entryCell` in `internal/boardengine/render.go` now stacks one code-span bullet per run-changing entry field the task sets,
between the brief and the run settings, in a fixed order:

- `pull_request: <on|off|merge-in>`, which decides whether the landing opens a pull request;
- `on_land: <key>`, the action a front task runs once it lands.

A field left empty adds no bullet, so an entry that sets neither renders as before.

## Design decisions

- **Recipe and profile get no bullet.**
  The board entry names them as examples, but the Running, Ready, layer and Draft tables already show both in the Recipe column, as `recipe · profile`.
  A bullet would repeat them.
- **Fields left out.**
  I checked every `Task` field.
  `notify_on_land` only asks the dobby to post a line once the run ends, and the dobby clears it after posting, so it doesn't change how the run goes.
  `isolated` and `depends_on` already decide the section and the After cell.
  `owner_machine` already shows in the Owner column.
  `front` only orders lym's own filing.
- **Line form.**
  Each line is `<board.json key>: <value>`.
  This is the same shape a run setting takes when it is a leaf directly under its module name.

## Tests

`TestRenderReadmeGolden`'s fixture now gives one draft task a `merge-in` pull request and an `on_land` key, and gives an isolated task `pull_request: on`.
The golden pins the bullet order: brief, then `pull_request`, then `on_land`, then the sorted settings.
The package doc in `doc.go` describes the new bullets.
