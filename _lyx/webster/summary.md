# Plan test-file path: refuse plain test-file paths under Edit

Batch 01 (`01-edit-test-file-check`) recorded `status: done` at head `fa8ea1c07df0be17dae6ec6e25800d94ba5b67a6`.
The fork's reply says its prompt file assigned cards 1 to 3, so it implemented all three as separate commits:

- `db5094b9d` — the plan gates refuse a plain test-file path under `**Edit:**`.
- `27c0593d4` — gate ways forward name a test file's members, never the file.
- `fa8ea1c07` — the plan stencil names new tests by handle and test neighbours under `**Uses:**`.

The scope guard listed one deviation: `internal/planparser/edittestfile.go`, a file outside the batch's declared targets.
It is informational and the batch recorded without warnings.
The plan-level verify gate runs after this turn ends.
