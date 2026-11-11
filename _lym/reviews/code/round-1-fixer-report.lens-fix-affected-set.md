```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Checked, each against the fix:
- **F1 (callers of the subject walk):** every caller of `ownerSubject` and `Covers` goes through `Watch.Covers`, which now passes the `listed` closure. The callers are `internal/beamcli/dobbydeps.go:143`, `internal/beamcli/ownerarchive.go:75` and `internal/orchcli/watch.go:188`. The change is in the one shared path, so no site was missed. `internal/dobbyengine/view.go:125` has its own `Covers` on an unrelated type.
- **F1 (twin walks):** `Owners.AdoptSubject` stops at `isBeamProject()` and `Owners.Effective` reads owners, not adopt cover. Neither shares the defect, so no twin needs the listing rule.
- **F5 (episode sites):** `archiveFailures` and `archiveFailing` are touched only in `handover.go` and declared in `loop.go:321`. `endArchiveEpisode` is called on both paths (`handover.go:50` and `:78`), and `Fail` is at `:70`. No other site is left.

Calls made:
1. `grep` for `ownerSubject(`, `.Covers(`, `NewWatch(` and `UnderAdoptWatch(` in non-test Go code: to find F1's callers, which my context lacked.
2. `grep` for `archiveFailing`, `archiveFailures` and `endArchiveEpisode`: to find F5's sites.
3. Read `internal/boardengine/owners.go` lines 125-200: to see whether `Effective` or `AdoptSubject` repeats the walk.
