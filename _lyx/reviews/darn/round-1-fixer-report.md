# Round 1 fixer-report

## Changes

All three findings were fixed as proposed, one commit each.

- F1 (`64f79546`, formatting follow-ups `bc3c7ec1` and `2aa21457`): `newRenameBoard` in `internal/boardengine/rename_test.go` gained the entries `exact` (body is the old slug), `starts` (body starts with it) and `ends` (brief ends with it).
  The `near` brief now also holds `old2` and `2old`.
  The wanted `Mentions` is `new, prose, exact, starts, ends` and the wanted store order includes the new entries.
  No test function was added.
- F2 (`f7165613`): `stepRename` in `internal/boardcli/verbs_test.go` makes its second rename address the entry by id, read from the `get` envelope, and asserts the resulting slug.
- F3 (`b2bef413`): the `Store.Rename` doc comment in `internal/boardengine/rename.go` states the refusals itself instead of pointing at `Board.Rename`.

`lyx gate test` passes for `./internal/boardengine` and `./internal/boardcli`.

## Gate repair

The gate reported a fixed-column-wrapped comment line in the `newRenameBoard` doc comment, `internal/boardengine/rename_test.go:21`.
`2aa21457` breaks that comment at its sentence boundary, one sentence per line.
`lyx gate test ./internal/boardengine` passes.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: LOW, the `wantMentions` line lost the space after `;`, a gofmt violation. Fixed as proposed in `bc3c7ec1`.

## Disputed

None.

## Deferred

None.
