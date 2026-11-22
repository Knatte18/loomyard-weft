```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Set searches I reran:
- **F12:** a word-match grep for `focusPath` across `internal` and `cmd`, excluding `burlerengine`, leaves only a comment and a parameter name in `internal/shedadapters/bouncer.go` (lines 1794, 1888, 1892, 1921).
  - Every call site now uses `FocusPath`.
  - The only other `round-%d-focus` literals are `round.go`'s own definition, which is the single owner, and test assertions that pin the name on purpose.
- **F13:** `harden-live-copies` now appears only in the new `reinterpret.go`, its test and the `configreg.go` registration; the template, `goReadGates` and `hardenshed` no longer name it.
  - `internal/hubreconcile/walk.go:509` reads each module's `Reinterpretations`, so the gate module's marker is snapshotted and restored on a failed walk without further changes.

Tool calls:
1. `grep -rnw focusPath` over `internal` and `cmd`, to enumerate remaining sites for F12.
2. A repo-wide grep for `harden-live-copies` and similar names, to enumerate remaining sites for F13.
3. A grep for `round-%d-focus` and the reinterpretation marker helpers, to find twin literals and other consumers of the marker set that my context lacked.
