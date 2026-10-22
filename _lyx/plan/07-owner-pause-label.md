# Card 7 — owner-pause-label

**Edit:**
- `internal/pauserecord#Coverage`
- `internal/pauserecord#Record.CoverageOf`
- `internal/pauserecord#TestCoverageOf`
- `internal/pauserecord/doc.go`
- `internal/tenderengine#PauseRequest`
- `internal/tenderengine#Pause`
- `internal/tenderengine#TestPause`
- `internal/tenderengine/doc.go`

**Create:**
- `plan:internal/pauserecord#OwnerCaller` -> `const OwnerCaller = "owner"`

**Intent:**
D4's fixed label for owner pauses, so `lyx adopt` lifts exactly those and InnerRun tells them apart.
`OwnerCaller` is the `Caller` every owner pause entry carries.
`Coverage` gains `OwnerHeld bool`, which `Record.CoverageOf` sets when a live entry of the slug's own scope carries `OwnerCaller`, whatever entry is the oldest covering one.
`PauseRequest` gains `Caller string`: when set, the entry `Pause` adds carries it instead of `Deps.Caller` (the session-skip rule still reads `Deps.Caller`); a scope that already has a live entry adds none, as today, so a caller that finds an unlabelled entry standing adds no owner pause.
`pauserecord`'s `doc.go` names the label and `OwnerHeld`, and `tenderengine`'s doc's Pause section says a request's `Caller` replaces the deps' caller on the entry.
`TestCoverageOf` gains assertions, beside its existing ones over one record: a slug entry with `OwnerCaller` covers with `OwnerHeld`, a hub entry older than it still leaves `OwnerHeld` set, an entry with another caller leaves it unset.
`TestPause` gains rows: a request with `Caller` adds an entry carrying it, and over a live entry of the scope adds none and reports `already_paused`.

**ImpactSummary:** two leaf-level additions in pauserecord and tenderengine; existing callers leave `Caller` empty and keep today's entries.

**Commit:** 7: pause: a fixed caller label for owner pauses
