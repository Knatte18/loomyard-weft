# Board README: stack each task's run settings under its title

The board README now shows a task's `run_settings`, so the review budget a task will run with can be read there.

## What changes

- `entryCell` in `internal/boardengine/render.go` appends one `• ` bullet per run setting after the brief, each a code span `<module>: <key path> <value>`, for example `darn: review_max_bounces 1`.
- A key nested below the module renders as its dotted path, such as `darn: models.writer opus`.
- A string value renders bare; any other value renders as its compact JSON, so a stored `1` shows as `1` and a list as `["a","b"]`.
- The lines are sorted, so the output is deterministic despite Go map order.
- No column is added, and an entry without `run_settings` renders exactly as before.
- `internal/boardengine/doc.go`'s `# README` section and `render.go`'s file header describe the setting lines.

## Design decisions

- The setting lines go in the entry cell shared by every README table, so a note or done entry that carries settings shows them too; one cell builder serves every table.
- Each setting line is a code span so it reads apart from the brief bullet beside it.
- Values are not backtick-escaped inside the code span: config keys and values are not expected to hold a backtick.
  A pipe is escaped like every other cell text.

## Tests

`TestRenderReadmeGolden`'s fixture gives three tasks run settings: two keys in one module after a brief, a nested key whose value holds a pipe beside a list value, and a single key on a task with no brief.
The golden pins the sorted bullets, the dotted path, the JSON value and the escaped pipe.
No top-level test is added.
