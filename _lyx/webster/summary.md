# Twill for Plan-Write: fabric vocabulary check and plan producer key

Batch 01 (cards 1 to 5) reached `done` at `f1d4d69f7`.
- The fabric-vocabulary predicates moved into the stdlib-only `internal/fabricvocab`.
- `planparser.CheckFabricVocabularyName` was added and wired into the plan index, rework and dispatch validation, with spec and help text.
- The twill directive takes its spawn point, rejection and severity rules and the reviewed-files phrase from its host.
- The `twill-plan` fan and its four lenses were added to the burler template.

Reported deviations for batch 01, all docs or help text:
- `contracts/specs/loom-plan-spec.md`
- `internal/loomcli/validate.go`
- `internal/planglyph/doc.go`
- `internal/twilldirective/doc.go`
- `internal/planparser/doc.go`

Batch 06 (cards 6 to 9) reached `done` at `404444985`, with no deviations.
- `plan_producer` was added to `loom.yaml`, `single` by default.
- The plan ready marker is removed before every fresh Plan-Write spawn.
- `lens-review.md` is admitted in the plan directory.
- Plan-Write renders the twill directive and authorizes forks under `twill`.

Both forks reported that the card and batch gates list `./internal/planparser_test`, which is not a package.
`lyx gate test` reports `[setup failed]` for that path, and every real package passed.
The fix is a plan edit, which is outside Master's role.
The forks' friction notes are under `_lyx/loom/friction/`.
