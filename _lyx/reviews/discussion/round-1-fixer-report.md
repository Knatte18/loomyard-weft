# Fixer report — discussion round 1

Every finding's premise held against the code; all eleven are fixed in `_lyx/discussion/decision-record.md`, with the support log's Turn 3, Q2, Q4, Q5 and a Round 1 entry updated to match.

## Changes

- F1 (BLOCKING), fixed as proposed and widened: D4's gitexec bullet has gitexec log every exec-core spawn, the probe included, through `internal/logger`, since `go list -deps ./internal/logger` no longer reaches `internal/gitexec`; the `internal/gitexec/gitexec.go` allowlist entry in `cmd/lyx/spawnobservability_test.go` goes, since its import-cycle reason is false. An unparseable git line gets a Warn as in reed. The Constraints bullet says the same.
- F2 (MEDIUM), fixed as proposed, option (b): before `--skip`, `CherryPick` compares the picked commit's tree with its first parent's tree; equal trees return an error with the pick in progress, so `dropSeedCommits` aborts with `errDropReplayFailed` as on git 2.53. The probe count, bound, Constraints bullet and a new acceptance criterion follow.
- F3 (MEDIUM), fixed as proposed: D3 and the acceptance criterion use the `S` loop's order, name on tmux 3.4 and id on newer tmux; Constraints name the difference, and D5 names it as its one same-behaviour exception.
- F4 (MEDIUM), fixed as proposed, by the first example: the marker is a `Paused` call for the receiver's strand, recorded per strand by `creelfake.Delivery`. D6 names how it excludes the sweep, `Store.post`, the other verbs and other workers, the interrupt path as the other pause reader, dormant receivers, and the multi-reader subtests it covers.
- F5 (MEDIUM), fixed as proposed, option 1: gitexec exports its check and `planglyph.Delta` calls it before `repo.DeltaGit`; the Rationale names the quarry path. A bound adds `internal/gitkit`'s unchecked git spawns, test plumbing no `lyx` command reaches.
- F6 (MEDIUM), fixed differently: the review's first option hangs the cache off `TmuxCmd`, but `TmuxCmd` is a value type that `NewTmuxCmd` and `newTmuxCmdForSocketPath` build afresh at each call site (`internal/reedengine/overlay.go:49-57`), so a cache on the value would probe again for every new value. Instead a hook-built `TmuxCmd` carries its own admitted result and never touches the process cache, keeping `lock_test.go`'s and `overlay_test.go`'s counts, and a test injecting into a process cache runs serial, names the global state and resets it.
- F7 (MEDIUM), fixed as proposed, with scope cut: the tmux criterion covers the runner, the server spawn and reedengine's exported check; `planglyph.Delta` and `reedcli`'s attach are pinned as calling the exported check, since an `export_test.go` cannot reach another package's cache. The Windows case is a Windows-only test file.
- F8 (MEDIUM), fixed as proposed, second option: the slowed run is the Plan's one-off diagnostic edit that never lands, the criterion asks that the tests wait for the lock signal, and the seam's bound names its lock sites in `merge.go` and `mergelifecycle.go`.
- F9 (LOW), fixed as proposed: the mark is set wherever `pinGeometryOptionsLocked` runs for a prime geometry, at boot and in the attach pre-flight (`attach.go:97`, `lifecycle.go:582`); the criterion adds attach.
- F10 (NIT), fixed as proposed: D3's rejected-alternative line is deleted, and D6's host sentence moved to the support log's Turn 3.
- F11 (NIT), fixed as proposed: an at-floor Apple line is admitted and a below-floor Apple line refused.

## Self-check

- fix-consistency MEDIUM, the gitexec allowlist entry is left with no valid reason: fixed as proposed (see F1).
- fix-consistency MEDIUM, D5's same-behaviour rule contradicts D3's order difference: fixed as proposed, D5 names the exception.
- fix-consistency LOW, `planglyph` and `reedcli` tests cannot inject into another package's cache: fixed with own change, the criterion pins their call of the exported check.
- fix-consistency LOW, support log Q2 stale against D2's tree comparison: fixed as proposed.
- fix-consistency LOW, support log Q4 stale against the planglyph call: fixed as proposed.
- fix-claims MEDIUM, `Store.Interrupt` is not the only other pause reader, an interrupting inject also reaches `interruptTarget`: fixed as proposed, D6 names the store's interrupt path and both callers.
- fix-way-forward MEDIUM, cross-package injection unreachable: duplicate of the fix-consistency LOW, fixed there.
- fix-way-forward MEDIUM, "no probe runs on Windows" cannot run on this host: fixed with own change, a Windows-only test file pins it, since an injectable OS seam would be production code for a test.
- fix-way-forward MEDIUM, a stale-wake probe can deliver the post and retire the worker, so the marked probe never comes: fixed as proposed in substance; a positive check waits for the outcome it asserts and a negative check waits for the marked probe, and the acceptance criterion says so.
- fix-proposal MEDIUM, gitexec's main spawn left unlogged once the import is legal: duplicate of the fix-consistency MEDIUM, fixed there.
- fix-proposal LOW, Windows criterion untestable here: duplicate of the fix-way-forward MEDIUM, fixed there.

## Disputed

None.

## Deferred

None.
