# Card 2 — tendercli-export-hubdeps

**Rename:**
- `internal/tendercli#hubDeps` -> `plan:internal/tendercli#HubDeps`

**Intent:**
Export the hub wiring of the pause engine so `internal/hibernatecli` (card 5) fills `tenderengine.Deps` the same way `lyx pause` does, instead of re-deriving the shuttle-backed pairs, the prime, the network operation and parent lock readers and the orch probes.
Rename the function with an AST-aware rename and change nothing else: its four callers in `internal/tendercli/cli.go` (`pauseCmd`, `resumeCmd`, `reviveCmd`, `statusCmd`) follow the rename, and its doc comment opens with the new name.
Its signature stays `(location *lyxcwd.Location, strand string) (tenderengine.Deps, []string, error)`; strand becomes the engine's `Caller`, which card 5 sets to the hibernate daemon's caller label.
A pure rename: `tendercli`'s existing tests cover it.

**Commit:** 2: tendercli: export HubDeps for the hibernate wiring
