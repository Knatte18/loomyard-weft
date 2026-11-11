---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/gatecli/testmapaudit.go:64
    summary: the audit round never reads HEAD before an audit package job, though D8 has a pass read HEAD before each package job
    fix: In `audit` in internal/gatecli/testmapaudit.go, call `r.readHead()` at the top of each iteration of the `for _, pkg := range slices.Sorted(maps.Keys(tests))` loop, after the `r.ctx.Err()` check and before the `r.headMoved` check, and return its error. Add a row to the HEAD-move coverage in internal/gatecli/testmap_integration_test.go where HEAD moves after the last queued job and before the audit, asserting no audit job builds.
  - id: F2
    severity: LOW
    class: design
    location: internal/gatecli/testmap.go:170
    summary: registerUpkeepPass lists the live RoleTestMap records and then registers, so two --upkeep calls starting together can both pass the refusal
    fix: In `registerUpkeepPass`, hold a lock around the List and Register pair, for example a write lock taken through the `lock` package at a path under the run-process directory named by `lymdirs`, released once the registration is written. Keep the refusal message and the return values as they are.
  - id: F3
    severity: LOW
    class: consistency
    location: internal/gatecli/testmappackage.go:250
    summary: a package whose job yields and resumes enters the built list and the per-package measures once per resumed job, so the report names it twice
    fix: In `runRounds`, append to `r.built` only when the package is not yet in it (for example `if !slices.Contains(r.built, p.job.pkg)`), and in the deferred append in `mapPackage` add the resumed job's build and run seconds and tests to the package's existing entry in `r.measures` instead of appending a second one. Cover it by a row in `TestGateTestMapPassYields` asserting one built entry and one measure for the resumed package.
---

### [LOW] The audit round never reads HEAD before an audit package job

**Location:** internal/gatecli/testmapaudit.go:64

**Issue:** D8 has a pass read HEAD before each package job and end when it moved.
`mapPackages` does so before each queued job, but `audit` only tests `r.headMoved`, which nothing sets between audit jobs until a round ends.
A landing between the last queued job and the first audit job, or between two audit jobs, costs a full coverage build before `endRound` finds it and discards the round.
It is bounded to one wasted job per landing, so the severity is LOW.

### [LOW] registerUpkeepPass is not atomic

**Location:** internal/gatecli/testmap.go:170

**Issue:** D8 has the verb refuse to start while another registered process lives.
`runproc.List` and `runproc.Register` are separate steps with no lock between them, and `Register` has no exclusivity of its own.
Two calls that list before either registers both start a pass.
The dobby starts at most one pass, so only hand callers racing each other reach it, and the build lock keeps the two off the same package; hence LOW.

### [LOW] A resumed package is reported twice

**Location:** internal/gatecli/testmappackage.go:250

**Issue:** After a yield, `measureJob` runs `mapPackage` again for the same package with a new `packageRun`, whose `wrote` starts false.
Each resumed job therefore appends the package to `r.built` and appends a second `PackageMeasure` through the deferred append.
The report's `built` list then names the package twice and `packages` shows it as two measures, one per slot hold.

## Rejected

- Round-level discard on a yield drops the tests of a parallel round that had finished before the yield.
  D10 states the write granularity as the round, so this matches the answer key.
- The yield poll does not back off while a job runs.
  D9 and PATTERN-daemon-wakeups ask for backoff in the wait for a slot, which `waitForSpareSlot` does; the poll is once a second, and `pattern/PATTERN-daemon-wakeups.md` lists it.
- A named call also ends when HEAD moves.
  D10 reads HEAD after each round of every job, which a named call also runs; no decision text exempts it.
- `TestNextRound` and `TestOrderJobs_TakesTheCostliestPackagesFirstThenImportPathOrder` are new public-surface-adjacent tests, but the redundancy report lists them as not judged and each pins a behavior no other test covers.
