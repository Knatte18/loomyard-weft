# Round 3 fixer report

All three findings are fixed; each is a `PATTERN-comment-line-breaks` correction to comments the diff added, with no code change.
`go build`, `go vet -tags integration` over the touched packages and `go test` over them pass.

## Fixed

- F1 (`c00715c0b`, `7173def14`): `add.go`'s live-pair list is now a sentence break plus two clauses split at a semicolon, and `remoteleftover.go`'s header keeps both predicates on one line.
  The `remoteleftover.go` header edit landed in the F2 commit because the file also carries F2's fix.
- F2 (`7173def14`): the clauses of `probeWeftLeftover`'s doc, `ReadConfigChanges`'s `so` clause and `doc.go`'s adopted-pair sentence each break at their independent-clause boundaries.
- F3: the added test comments listed in the finding break before their second clause, in `fabricengine`, `fabriccli`, `landingshed` and `loomcli` test files.

No test was added: none of the findings is a coverage gap.

## Deferred

None.
