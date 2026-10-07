```yaml
round: 3
fixed: [F1, F2, F3]
deferred: []
```

## What changed

Each finding is one commit on `run-pushes`, none pushed.

- **F1** (BLOCKING, consistency): broke three comments in `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide` at their independent-clause boundaries, in `internal/fabricengine/pushanchored_integration_test.go`.
  These are the failed-fetch step's `, and a remote that declines once would accept a retry`, the unborn-side step's `, and the records side still pushes`, and the both-sides-failing step's two comma-joined clauses.
  Wording is unchanged apart from the break points and one added `and`.
- **F2** (LOW, consistency): `commitStatusDeps.Push` in `internal/loomcli/wiring.go` now says `A push lock that stayed busy names no side.`
  The previous sentence claimed every error from taking the push lock names none, which a lock-directory creation error contradicts: its text says `weft`.
- **F3** (NIT, consistency): the test's doc comment now lists the rejected-push-then-failed-fetch behavior at its place in the walk.

No test was added: none of the findings is a coverage gap.

## Verification

- `go build ./...` and `go test ./...` pass.
- `go test -tags integration` over `internal/fabricengine` and `internal/loomcli` passes.
- The change is comments only, so the other integration packages in the plan's verify set are unaffected.

## Deferred

None.
