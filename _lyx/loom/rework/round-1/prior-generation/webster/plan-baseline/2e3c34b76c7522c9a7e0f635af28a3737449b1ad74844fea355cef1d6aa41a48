# Card 11 — slug-taking board verbs state the limit

**Edit:**
- `internal/boardcli#Command`
- `internal/boardcli#storeVerbs`
- `internal/boardcli#intakeCommand`
- `internal/boardcli/help_test.go`
- `internal/boardcli/doc.go`
- `plugins/ly/skills/board/SKILL.md`

**Uses:**
- `plan:internal/boardengine#MaxSlugLength`

**Intent:**
The `Long` help of every board verb that takes a slug states the limit, formatted from `boardengine.MaxSlugLength` so help and validation cannot drift (D8): `get`, `upsert`, `upsert-batch`, `set-status`, `remove`, `merge`, `promote`, `set-deps` and `intake import`.
Each such `Long` becomes a `fmt.Sprintf` over the constant, with one shared sentence such as `A slug is at most %d characters.`.

Tests, untagged in `help_test.go`: for each verb above, `--help` contains the sentence formatted from `boardengine.MaxSlugLength`, never a literal number.

`doc.go` and `SKILL.md` name the limit by pointing at `--help` rather than repeating the number.

**ImpactSummary:** help text of the slug-taking board verbs; no behaviour change.

**Commit:** 11: slug-taking board verbs state the limit
