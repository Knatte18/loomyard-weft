# Card 11 — config-package-docs

**Prosa:**
- `internal/configreg/configreg.go`
- `internal/configsync/configsync.go`

**Intent:**
Update the package doc comments above the package clauses of `configreg.go` and `configsync.go` (D10), which no earlier card could target beside the members it changed in those files.
`configreg`'s says the registry also lists every registered engine's config module, reached through `shuttleengines` so `configreg` imports no concrete engine, and declares the cross-module key-move table.
`configsync`'s says every reconcile entry point, and run-settings preparation through `configreg`, applies the key-move table to each layer as its first step, before any module of the layer is seeded or reconciled.
Comment text only; no code changes.
