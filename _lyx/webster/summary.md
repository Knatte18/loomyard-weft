# Plan gate fixes: typed delete-before-reference and validate-plan --overlay

Batch 01 (`01-shared-reference-delete-order`) recorded `status: done` at head `5acf01885d27bd745dc5e046c4d2a154af91e437`.

The batch covered the plan's six cards:

- `delete-before-reference` reads the same reference computation as `caller-uncovered`, over Go code only.
- Webster's batch recording and `recover-batch` count only blocking delete-before-reference findings.
- `planparser` parses and rewrites a plan through an in-memory overlay.
- Canonicalization, its re-parse and the done-card comparison read through the plan's own source.
- `validate-plan --overlay` reads proposed plan contents from stdin.
- The Plan-Review rubric validates proposed fixes with `--overlay` and names a comment's enclosing function.

## Deviations

`internal/planparser/parse.go` changed outside the declared file-ops.
The overview-reading step moved out of `ParsePlanFrom` into a helper, so the overlay constructor reads the Card Index the same way.
`record-batch` also returned informational scope warnings for symbols touched outside the batch's target glyphs, in `internal/loomcli`, `internal/planglyph`, `internal/planparser`, `internal/planindex` and `internal/websterengine`.

## Gate-step note

The fork reported that two card gate steps, `./internal/websterengine_test` and `./internal/planparser_test`, name external test packages rather than directories and exit 1 with `directory not found`.
It reran the batch gate without them and the packages pass.
It recorded this in a friction note.

## Verify gate fixes

The verify gate's first attempt failed on `TestEnforcement_FabricVocabulary/tree-scan`: `CLAUDE.md` and `README.md` held a bare weft/warp token.
The token came from the records repository's name `loomyard-weft`, and from branch names in `README.md`, both from an earlier README commit rather than from batch 01.
The fixer made one commit, `a2618efa4`:

- `internal/lyxcwd/enforcement_test.go` removes `loomyard-weft` from the two root files before the bare-token check, since both must spell it to link the repository.
- The `README.md` "Runs in flight" bullet drops the branch names and now reads "a records branch per running task, and one for `main`".

The fixer ran `lyx gate test --tags integration ./internal/lyxcwd`, which passes, and did not run the full plan verify.
