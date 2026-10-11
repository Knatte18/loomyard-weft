# Fixer report — plan, round 2

## Fixed

- **F1** (LOW, consistency): confirmed against `internal/fabricengine/unwire.go` (file header "Unwire always removes every fabric junction present on disk", `JunctionsRemoved`'s "actually present and removed", the bare `GitExclude` note, the `Unwire` doc comment) and `internal/fabriccli/unwire.go` (file header "full deactivation", `runUnwire`'s "removing every on-disk fabric junction").
  Card 6's `Docs:` sentence now also names the `Unwire` doc comment, `UnwireVerbResult`'s doc comment and its `JunctionsRemoved` and `GitExclude` field comments, `runUnwire`'s doc comment, and both `unwire.go` file headers, each stating the dry-run behaviour.
  No target change: the symbols are already Edit targets and the headers sit in their files.
- **F2** (LOW, scope): confirmed at `internal/configcli/reconcile_integration_test.go:92`, whose dry-run step asserts the `loom` entry omits `migrated` "without a migration".
  Card 5 now lists `internal/configcli#TestConfigReconcileInGitRepo` under `**Edit:**` (the file is `package configcli`), and its Intent retargets the assertion to the `reed` entry, which keeps no hook and which the same step already checks, with a message naming reed.
  The plan's `go test -tags integration ./...` already runs it.

## Disputed

None.

## Deferred

None.
