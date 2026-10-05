# Card 1 — Mark hub-wide modules in the config registry

**Edit:**
- `internal/configreg#Module`
- `internal/configreg#Modules`
- `internal/configreg/configreg.go`
- `internal/configreg/configreg_test.go`

**Intent:**
Add a `HubWide bool` field to `configreg.Module` (D2), documented as: the module describes a hub-level fact, so its one file lives at `configengine.ConfigFile(<BoardDir>, name)` and is never read or written in a worktree's `_lyx/config/`.
Set it on `fabric` and `board` in `Modules()`, and on no other entry (D1).
Update the file's header comment, which today calls the registry a list of "available config modules (board, fabric)", to name the hub-wide marking as the registry's second role.

This card only declares the field; no reader branches on it yet, so behaviour is unchanged.

Coverage: add `TestModules_HubWide` beside `TestModules_SeedOnly` in `configreg_test.go` that pins `HubWide` exactly as that test pins `SeedOnly`: true for `fabric` and `board`, false for every other registered module.

**ImpactSummary:** One new struct field set on two registry entries; no caller reads it in this card.

**Commit:** 1: configreg: mark fabric and board hub-wide
