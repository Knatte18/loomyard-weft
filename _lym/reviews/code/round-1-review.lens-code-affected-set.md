```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. I list, per change, what reads, calls or asserts it, then give my findings.

**Per change, the set found**
- `NewSelector(config, target)` signature and removal of `SelectorConfig.WorkingFiles`: the only production caller is `internal/gatecli/coveragescope.go:302`, and the only test caller is `internal/testcover/testselect_test.go:470`. Both are updated. No `WorkingFiles` reference is left in code or docs. Covered.
- `MissOwnFiles` removal, plus the renames `workingHash`, `readWorkingFile`, `ownFilesMiss` and `linkedChanges`: a repo-wide grep finds no leftover reference. Covered.
- `Selection.Recorded` now holds only passing entries: the only consumer is `gatecli.coverageUnion`, which was updated to stop checking spawn marks. `internal/testcover/verdict.go:126` reads `Spawn` for its own verdict and is unaffected. Covered.
- `MapTest.Outcome` is a new `omitempty` field. Old maps read as passing, and nothing in the diff rejects an unknown key, so `PATTERN-tolerant-records` holds. The diff adds no writer for it; the summary assigns that to `testmap-upkeep-plan`. Covered.
- Environment stamp: `stampEnvironment`, `mappedTierConfigs` and `StampModuleFiles` are shared by the map build (`testmappackage.go:405`) and the gate scope. `testRequest.hubEnv` is set in `run.go:81`, and `testmap.go`'s `prepare` now uses `mappedTierConfigs`. Covered.
- Stamp and function-set map shape, `MapFormatVersion` unchanged: the summary says D3 needed nothing more. This matches the diff, which leaves the format alone. Covered.
- Docs: `internal/testcover/doc.go` and `internal/gatecli/doc.go` are updated to the three-rule union, the target argument, the stamp miss and the test-file rule. Covered.
- `PATTERN-fabric-boundary`: the commit target is read only through the `CommitReader`, which is `fabricengine.Fabric` in production. Covered.
- Declared PATTERN changes: the directive declares none, and the diff touches no `PATTERN.md` or `pattern/` file. Nothing to land.
- Commit messages: neither commit carries a `Pattern-Departure:` trailer, so there is nothing to judge.

**Findings:** none.

**Tool calls (2)**
1. `grep` for `WorkingFiles`, `MissOwnFiles`, `NewSelector(`, the renamed helpers and `.toolchain` across Go, markdown and yaml: the manifest listed no callers.
2. `grep` for `Selection{`, `.Recorded`, the `Spawn` flags and `testcover.Selector` outside `testselect*`: to find other consumers of `Selection.Recorded`.
