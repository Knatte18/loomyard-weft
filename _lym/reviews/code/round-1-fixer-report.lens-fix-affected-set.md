```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the finding searches for F1, F2, F8 and F9 and found one stale sibling.

Findings:

- **LOW**: the header of `internal/loomcli/stencilpins.go` (line 2) now understates the warn path.
  - Text: "On a resume it warns the run's parent orch of each pinned stencil whose installed board copy changed since."
  - Since F8, `startSuccessFields` also calls `warnChangedStencils`, so a start that puts a driver to work warns too. `doc.go:320` and both long helps were updated, but this file header was not.
  - Evidence: `internal/loomcli/start.go:500` (`withChangedStencils(fields, c.warnChangedStencils())`).

Searches by finding:

- **F2 (every place that promises what `lym stencil diff` shows):** the sites are `stencilpins.go:36` and `:65`, `start.go:529`, `resume.go:259` and `doc.go:323`. All carry the new wording. The other `lym stencil diff` mentions, in `friction.go`, `burlerengine/prompt.go`, `stencilstore/reconcile.go` and `stencilcli`, make no change-since-run-start promise, so nothing was missed.
- **F8 (callers of `runDriverSpawnAndWait`):** the only production caller is `start.go:634`. It uses both results, and every `return false` inside the function became `return false, false`. The build is green. The "every successful start" sentence existed only in `doc.go` and the start long help, and both were updated. `resume.go` calls the check only on the two wake paths, so it matches.
- **F1 (commit pathspec sites):** `bootstrapCommitPaths` has one production caller, `sharedbootstrap.go:184`, and the pins are now included whenever the file is on disk. `statusCommitPathspec` in `wiring.go:113` does not carry the pins file. That does not matter here, because every `start` and `step` re-enters `seedAndCommitBootstrap`, which now commits it.
- **F9 (resume success sites):** the only two warn sites, the spawned-Go-driver path and the woken-parked-driver path, both go through `succeedAfterWake`. The `succeed` paths that remain (done, already running) warn of nothing, as intended.
- **Not checked by a search:** F3, F5, F6 and F7 are coverage gaps with no set premise.

Tool calls used:
1. A grep across `*.go` and `*.md` for the diff-promise and "every successful start" wording, to cover every member of F2 and F8. My context held only the files I had read.
2. A grep for the callers of `runDriverSpawnAndWait`, `bootstrapCommitPaths`, `warnChangedStencils`, `withChangedStencils` and `writeStencilPins`, plus the status commit pathspec sites, to cover F1, F8 and F9.
