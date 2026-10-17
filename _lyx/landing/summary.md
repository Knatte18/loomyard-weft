# Show each task's run settings in the board README

The board README now shows a task's `run_settings`, so the review budget a task will run with can be read there, for example `{"darn":{"review_max_bounces":1}}`.
The tables were already hard to read, so the settings add no column: they stack in height inside the existing entry cell.

## What changes

- Each setting is a `• ` bullet in the entry cell, after the title and the brief, rendered as a code span such as `` `darn: review_max_bounces 1` ``.
- The entry cell is shared by every README table, so Running, Ready, layer, Notes and Done rows all show their settings.
- A task without `run_settings` renders as before.
- `internal/boardengine/doc.go` describes the settings bullets in its README section.

## Design decisions

- One line per leaf setting.
  A key nested below the module renders as its dotted path, such as `darn: models.writer opus`.
  Lines sort by module, then by key path, so the output is deterministic despite Go's random map order.
- A string value renders as itself and any other value as compact JSON, with HTML escaping off, matching what `lyx board get` shows: `1`, not `1.0`, and `["a","b"]` for a list.
- `codeSpan` fences with a backtick run longer than any in the text, so a value holding backticks cannot break the span.
  A pipe in a value is escaped by the existing cell escaping, so a row stays one table row.
- Out of scope: the design-doc header and `lyx board list --text`.

## Tests

`TestRenderReadmeGolden`'s fixture and golden carry run settings on some entries, covering a brief with several settings, a task with settings and no brief, and a nested key.
No new top-level test was added.

## Check by hand

Nothing outstanding; run `lyx board` against a board with settings and read the rendered README once.
