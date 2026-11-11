# Let promote set depends_on so a released Sketch waits for its deps

Promoting a Sketch with recipe `raddle` releases the task in the same write.
The parent's dobby can start that run before a later `lym board set-deps` lands, so the task ran with its dependency unfinished.
`lym board promote` now takes an optional `depends_on`, written in the same write that releases the task.

```
lym board promote '{"handle":"S3","depends_on":["other-task"]}'
```

Closes #107

## Changes

- `boardengine`: `Board.PromoteTo` and `Store.promoteTo` take a `dependsOn` list.
  A non-nil list replaces the task's `depends_on` in the promoting write.
  `PromoteTo` also returns the siblings the write created, through `writeReportingSiblings`.
  `Board.Promote` is unchanged: no recipe, no `depends_on`.
- `boardcli`: `promote` accepts `depends_on` beside the slug, id or handle selector, and prints the sibling notice.
  The list parsing moved out of `set-deps` into `dependsOnList`, which both verbs share.
  A null `depends_on` reads as an empty list, as in `set-deps`.
- The `board` skill (`plugins/ly/skills/board/SKILL.md`) teaches giving a Sketch's dependencies in the promotion.
- The `boardcli` and `boardengine` package docs describe the new field.

## Design decisions

- An entry that is already a task is returned unchanged, as before, and nothing is written.
  If the `depends_on` given differs from its own, `promote` refuses and names `lym board set-deps`.
  An equal list is a no-op.
  An empty list on a task that has dependencies is a difference, so it is refused too.
  This keeps `promote` from silently dropping a dependency change it cannot make.
- `set-deps` after a release is not changed.
  The window between release and a later `set-deps` is still open for callers that use it;
  the skill and help text now point to `promote` as the way to avoid it.
- The sibling notice that `set-deps`, `upsert`, `upsert-batch` and `merge` print now comes from `promote` too.
  `intake import` writes no `depends_on` and is unchanged.

## To check by hand

- No regression check ran for this task, and nothing was run by hand beyond the unit tests added in `boardcli` and `boardengine`.
- `plugins/ly/skills/board/SKILL.md`: the skill gave no `promote` `depends_on` example for a Sketch.
  The diff adds one; confirm the wording fits the surrounding section.
- `promote` sibling notice: the CLI and engine tests now pin it
  (`internal/boardcli/cli_test.go`, `internal/boardengine/siblings_test.go`).
  Confirm the notice text asserted in `cli_test.go` is the wording you want.
