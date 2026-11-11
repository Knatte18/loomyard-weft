```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens. I checked each finding whose premise is a set of sites.

**Findings: none.**

Findings I searched for set coverage:
- **F3, sites that append to `r.built` or `r.measures`:** the fix covers every site. There are three: the `runRounds` guard and the `addMeasure` call, and `write()` at `testmappackage.go:740`.
  - `write()` appends to `r.built` without the `slices.Contains` guard. A job reaches it only before any round is written, so the package cannot already be in `built` from a resumed job. It is not a defect.
  - `exitRecord` at `testmap.go:696` only reads `r.measures`.
- **F1, sites that read HEAD before a package job:** the three job loops are `mapPackages`, `audit` and the resume wait in `testmapyield.go:79`. The fix adds the read to `audit`, which was the only one missing it. `endRound` reads HEAD after each round.
- **F2, sites that register the upkeep pass:** `registerUpkeepPass` is the only registrar inside `gatecli`, and the lock now covers its check and registration. `beamcli/dobbydeps.go:631` also writes a `RoleTestMap` record, as a child record for the pass it starts. That registration is outside the lock, but it is the dobby's single start the review set aside (the review says the dobby starts at most one pass), so I do not count it as a gap.

Findings left unchecked: none.

Tool calls:
1. `grep` over `internal/gatecli` for `r.built` and `r.measures` writes. It covers F3's set of sites, which my context did not hold.
2. `grep` over `internal` for `readHead()`, `mapPackage(` and `RoleTestMap`. It covers the F1 and F2 sets, which my context did not hold.
