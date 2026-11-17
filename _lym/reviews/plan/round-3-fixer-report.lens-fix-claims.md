```yaml
lens: fix-claims
outcome: reported
```

Task: check that the fix's new claims hold against the code at HEAD (fix-claims lens).

no findings

- Card 1: "`inert`, whose one reader was `headSelector`" holds. The only read is `internal/gatecli/testmapplan.go:148`, which sits inside `testMapRun.headSelector` (starts at line 124).
- Card 1: "`prepare` … drops … the `r.inert = cfg.Inert` assignment" holds. The only write is `internal/gatecli/testmap.go:363`, which sits inside `testMapRun.prepare` (starts at line 335).
- Card 3: "the tier's `Parallel` as the config gives it, this machine's `tier_parallel` entry already applied" holds. `applyTierParallel` (`internal/gateslot/tiers.go:286`–299) lays the entry over the declared tiers. Load defaults `Parallel` to 1 (line 191) and refuses anything below 1 (line 236), so no value below 1 reaches `mapSideBySide`.

Tool calls:
- One grep for the functions that enclose `testmap.go:363` and `testmapplan.go:148`, and for every `r.inert` use in gatecli. My context had the line numbers but not which functions they sit in.
