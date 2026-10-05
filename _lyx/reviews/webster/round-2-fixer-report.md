# Round 2 fixer report

Every finding was a `PATTERN-comment-line-breaks` violation in a new or changed comment, fixed by breaking at the clause boundary or joining a fixed-column wrap, with no code change.
Each fix is its own commit, and the build and the tests of the touched packages pass.

- F1 — `internal/yamlengine/reconcile.go`: `findValueNode` doc on one line; `Reconcile` doc broken before "and nothing at or under the path".
- F2 — `internal/yamlengine/set.go`: `SetValues` doc and `setOpenMapEntry` doc broken one clause per line.
- F3 — `internal/yamlengine/doc.go`: broken before "and the result is idempotent".
- F4 — `internal/configengine/doc.go`: broken after both semicolons.
- F5 — `internal/configcli/configcli.go` and `doc.go`: `Command` comment and two `doc.go` lines broken at the clause boundary.
- F6 — `internal/boardengine/config.go` and `doc.go`: `Types`/`Labels` comment and the "older shape" line broken at the clause boundary.

## Deferred

Nothing.
