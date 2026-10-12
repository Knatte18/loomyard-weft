## Changes

- F1, fixed as proposed: `pauserecord.Update` now runs one `state.UpdateJSON[Record](path, path+".lock", ...)` call, so a rewrite keeps the keys of the replaced file that the record types do not declare.
  `writeAtomically` moved from `record.go` to `hold.go`, where `writeMarker` is its only user.
- `internal/state` is admitted in the import allowlist of `leaf_enforcement_test.go`, in the pauserecord section of `pattern/PATTERN-leaf-packages.md` and in `doc.go`.
  `internal/state` imports only the standard library, `internal/fsx` and `internal/lock`, so no cycle forms.
- `record_test.go` has one new `TestUpdate` row: an unknown top-level key and an unknown entry key survive `Update`.
- `lyx gate test ./internal/pauserecord` and `lyx gate test ./cmd/lyx` pass.

## Disputed

None.

## Deferred

None.
