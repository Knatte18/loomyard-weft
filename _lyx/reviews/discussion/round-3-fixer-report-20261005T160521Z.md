# Round 3 fixer report

All edits are in `_lyx/discussion/decision-record.md`.

- F1: the D6 Bound now states that files moved to `llm` from `smoke` or `integration` leave every automatic run, `cardGateCommand`'s integration run included, as intended.
  The retag acceptance bullet now covers retags into `llm` from either tag.
- F2: the closing acceptance bullet now includes `go test -tags tmux ./...`.
- F3: D2 defines unclassifiable: an unresolvable call, with production packages not followed and testkit and `hubforge` helpers scanned.
- F4: the retired-ID criterion is scoped to code comments, stencils, specs and docs, with the discussion artifacts exempt.

## Deferred

None.
