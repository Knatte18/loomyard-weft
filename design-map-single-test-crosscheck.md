# No lym check compares a map entry with a single-test coverage run

`map-single-test-crosscheck` · Note · enhancement, gate

Found by the map-tooling-trial.

Nothing in lym compares a map entry with a single-test coverage run.
The trial did it by hand: `lym gate test --wait ./internal/<pkg> -- -run '^<Test>$' -count=1 -coverprofile=<abs path>`, then counted covered blocks per file in the profile against the map's `blocks` of that test grouped by file.
That agreed for both `internal/durfmt` tests and for `TestOutputs`, `TestStoreValidatesLabelsWithVocabulary` and `TestWriteSiblings` of `internal/boardengine`.

Two limits:
- The map's blocks carry file, function and hash but no line range, so the check is by count per file, not block by block.
- The map's `funcs` list (functions of linked packages a test reaches) has no single-test counterpart in a statement profile, so it is unchecked.

Add a verb, or a `--map` option, that re-measures named tests alone and reports any block or function that differs, and give blocks a line range.
