```yaml
lens: fix-proposal
outcome: reported
```

Task: the fix-proposal lens, which challenges the reviewer's proposals that the fixer carried out as written.

Proposals challenged: F1 (staleness rule cut, Open risks, D5 Bound), F2 (host-admitted union member, D6 `-run` placement, D3 criterion), F3 (fresh dirty reads, Bound, D2 criterion), F4 (go:embed carve-out from `inert_paths`), F5 (predeclared-name miss).

**MEDIUM: F3's new D2 acceptance criterion states behaviour D2 does not have**
- **Changed text:** decision-record.md, Acceptance criteria: "D2: a linked package's file dirtied after the walk started and still dirty after a package's tests leaves that package without a map."
- **Problem:** D2 only says testmap "writes no map for that package, as it does when files changed at HEAD". In that HEAD case `mapPackage` calls `r.skip` and returns, so any earlier map stays on disk (`internal/gatecli/testmappackage.go:95-97`). The first dirty check also skips without touching the existing map, as the sibling criterion "leaves package B's map as it was" states.
- **Case:** B has an earlier map, and A is dirtied mid-walk. After the run B still has its old map, not "no map". The criterion fails against D2's own rule.
- **Fix:** the criterion should say testmap writes no new map for that package and leaves any existing one as it was.

**MEDIUM: F1 also cut the "lacks the marker" staleness clause, so rollout can wait up to 7 days**
- **Changed text:** D2, the deleted sentence "testmap counts a map stale … when it lacks the marker or …", and the Open risks bullet "every package needs one rebuild, which testmap makes by its present rules, before any selection".
- **Problem:** the parent's ruling covers the dependency-drift rule ("The dependency-drift rule D2 proposed goes into note testmap-upkeep"). The proposal deleted the whole sentence, including the marker clause, which is not dependency drift.
- **Effect:** under the present rules (`stale`, `internal/gatecli/testmap.go:456`), a map without the marker is fresh until its own files change or `test_map_max_age_days` passes. The default is 7 (`internal/gateslot/template.yaml:4`). D5 makes every such map a miss, so after landing `coverage` equals `diff` for up to 7 days per package unless someone names packages to testmap.
- **Fix:** the Open risks bullet should state this delay (the age limit, or a named-package testmap run). Otherwise the fixer should flag to the parent whether the marker clause falls under the ruling.

**LOW: F2's D3 rule says the tier's tag but does not name the gate's GOOS and GOARCH**
- **Changed text:** D3, second union member: "a file is admitted as `holdsTierTest` … admits one, through `go/build` `Context.MatchFile` with the tier's tag".
- **Problem:** `holdsTierTest` matches against `build.Default` plus the tag (`tierrun.go:400-401`), which is the host's GOOS and GOARCH. That is correct, but the record's D3 criterion uses GOOS while the rule names only the tag.
- **Assessment:** the reference to `holdsTierTest` makes the rule unambiguous, so this is no defect. No change needed.

No breakage found in F4 or F5:
- **F4:** a working-tree `//go:embed` change is already a `//go:` line miss, so reading the patterns at the map's `Commit` is enough. A new file under an embedded directory is caught by the directory-pattern clause.
- **F5:** with the predeclared-name miss in place, the dot-import case is covered, because the changed file is A's and A is in the linked set. An added non-predeclared name that collides through a dot-import fails to compile, so it cannot pass silently.

Tool calls:
- grep for `test_map_max_age_days`: to get the age-limit default behind the F1 finding. My context had `stale` but not the config default.
