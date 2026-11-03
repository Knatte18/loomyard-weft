```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set — task: rerun the set searches behind F1 and F2 and check the fix covers every member.

No findings.

**F1 (`commitStatusDeps` call sites):** the search finds exactly two sites that build the board-status deps, `wiring.go:319` (`wireLightweight`) and `wiring.go:957` (`wire`). The rewritten `TestWiredCommitStatus_WritesFinishedStatus` drives both, each with a motif row and a tapestry row.
- The other `loomCommitStatusDeps` users do not set `FinishedStatus`, so they are outside the premise: `commitrecords.go:111`, `drivewiring.go:255` and `:260`, and the `wiring_commitstatus_integration_test.go` calls. They drive the commit, park and push paths, not the board-status write.

**F2 (`LandDeps` fillers and seam lists):** the search finds every `LandDeps{` literal and every `Land.PruneBoard` listing, and each now has `ChildLanded`:
- Fillers:
  - `battencli/land.go:48` (production)
  - `battenshed/land_test.go:78`
  - `battencli/run_test.go:104`
  - `shedrecipe/fixture_test.go:275`
  - `testkit/envkit/envkit.go:525`
- Lists:
  - `shedrecipe/entries_batten.go` (validation list and its doc comment)
  - `shedrecipe/entries_batten_test.go` (zero-seam table)
  - `loomcli/wiring_seams_test.go` (`intentionallyNil`)
- The batten-side `TestWire_EverySeamFilled` needs no entry, because the seam is filled.
- No markdown file lists the Land seams.

**F3:** its premise is a list of return paths inside one function, not a set under one key, so I did not search for it. That makes it the one finding left unchecked.

Further tool calls (2):
1. `grep` for `commitStatusDeps(`, `loomCommitStatusDeps(` and `CommitStatus:` across `internal` — my context did not show every call site.
2. `grep` for `LandDeps{`, `Land.PruneBoard` and `PruneBoard:` across Go and markdown files — my context did not show every filler or list.
