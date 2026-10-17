# Decision record — Show each task run_settings in the board README

## Goal

Make a task's `run_settings`, such as the review budget it runs with, readable in the board README without adding a table column.

## Scope

- `internal/boardengine/render.go`: the README entry cell gains one line per run setting under the title and brief.
- `internal/boardengine/render_test.go`: the README golden fixture carries run settings on some entries.
- `internal/boardengine/doc.go`: the `# README` section describes the settings lines.
- Out of scope: the design-doc header, `lyx board list --text`, and any new README column.

## Decisions

### One line per leaf setting, in the entry cell

Decision: the entry cell (Task, Note or Entry column) stacks one `• ` bullet line per leaf setting after the brief, rendered as a code span `` `<module>: <key> <value>` ``, for example `` `darn: review_max_bounces 1` ``.
Rationale: the operator decided on no new column and stacking in height; the code span tells a setting line apart from the brief line.

### Nested keys join with a dot, lines sort by module then key

Decision: a key nested below the module level renders as its dotted path, such as `darn: models.writer opus`, and the lines sort by module, then by key path.
Rationale: `run_settings` nests as in the module's YAML, so a flat dotted path keeps each setting one line; sorting makes the output deterministic, since Go map order is random.

### Values render as JSON, strings bare

Decision: a string value renders as itself, and every other value as its compact JSON, so a number decoded as float64 renders as `1`, not `1.0`, and a list as `["a","b"]`.
Rationale: JSON is the stored form, so the rendered value matches what `lyx board get` shows.

### Every table shows the settings

Decision: the lines are added in the entry cell shared by every README table, so a done entry or a note that carries settings shows them too.
Rationale: one cell builder serves every table, and an entry with no settings renders unchanged, so there is no case to special-case.

## Constraints

- PATTERN-test-economy: extend `TestRenderReadmeGolden`'s fixture and golden; add no new top-level test.
- PATTERN-comment-line-breaks: Go comments one sentence per line.
- Docs land in the same commit: `internal/boardengine/doc.go` describes the README rows.

## Auto-mode assumptions

- The code-span form for a setting line is acceptable; the board entry's example shows the text without stating its markup.
- A setting value holding a backtick is not expected in config keys or values, so the code span needs no backtick escaping.

## Open risks

- A task with many settings makes its row tall; the operator chose height over width.

## Acceptance criteria

- A task with `{"darn":{"review_max_bounces":1}}` and no brief renders its entry cell as the bold title, then a line break and a bullet holding the code span `darn: review_max_bounces 1`.
- A task with a brief and two settings in one module renders the brief line first, then one line per setting, sorted by key.
- A nested setting renders its dotted key path.
- A task without `run_settings` renders as before.
- A pipe in a setting value is escaped so the row stays one table row.
