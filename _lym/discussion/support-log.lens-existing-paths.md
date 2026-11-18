```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced every existing path that writes or reads the state the record changes.

**Findings**

1. **BLOCKING — D7, failure triage lines.** Adding triage lines inside the failure report breaks the existing parsers of that report, and the record never mentions them.
   - Evidence: `gateslot.FailedTests` and `TierRunFailures` (`internal/gateslot/runlog.go`) read every line under the header.
     - A line indented exactly two spaces must be a `<package> <Test>:` line; any other two-space line makes `FailedTests` return `ok=false`.
     - Only a line indented four spaces counts as the failure's own output.
     - A cut report also gives `ok=false`. `failureReportLines = 80` caps the report, so the triage lines eat into that cap.
   - Who loses: `gateslot.FailedTestsCall` drives the next arrival's rerun of failed tests alone, and the Finalize flaky rerun (`PATTERN-gate-slot`) depends on the same parse. A triage line at the wrong indent, or one that pushes the report past the cap, silently disables both.
   - Fix: D7 should pin that a triage line is a four-space line under its test's line. It should also say whether triage lines count toward the line cap, or are dropped before the report is cut.

2. **MEDIUM — D3 and D2, fresh map build on a dirty tree.** The `--map` write refuses any package with a modified or untracked file, and refuses when HEAD moves.
   - Evidence: `testmap.go` `dirtyOwnedFile` / `dirtyModuleFile`, and `testmappackage.go` ("writes no map when ... a file the package ... owns is dirty").
   - D2 diffs "the worktree's files", which includes uncommitted edits. The rebuild that D3 reuses only measures clean, committed packages.
   - Where it survives: Darn-Write and Code-Burler-Fix run through `verifytree.Verify`, which refuses a dirty tree, so their trees are clean.
   - Where it breaks: a writer calling `lym gate test --gate batch` (D12 adds `reach` to `batch`) on uncommitted card work. Every changed package is listed as unmeasured, and the note silently falls back to stored maps that lack the new functions.
   - Fix: D3 should state this bound, or diff HEAD rather than the worktree's files.

3. **MEDIUM — D3 against D1, the step run in the prime.** `coverageMapsDir` (`internal/gatecli/testmap.go`) sends a `--map` measurement in the hub's prime to the hub's *shared* maps.
   - So D3's step, run by any gate call in the prime, rewrites shared maps from a gate. That contradicts D1's rule that "only the prime's `--map` run writes" the shared directory, and the D3 text "into the worktree's anchor".
   - It also leaves the index stamped stale until the next `--map` run. The in-memory fallback keeps answers right but slow.
   - Fix: the record should say the reach step's rebuild always targets the worktree's anchor (not `coverageMapsDir`), or that the step skips rebuilds in the prime.

4. **MEDIUM — D12, template defaults versus existing hubs.** This hub's `gate.yaml` spells every key and lists no `reports`. The record does not say whether `reach` reaches an existing hub through the template layer, or needs a `hubreconcile` reinterpretation under `PATTERN-config-layers`.
   - Evidence: `_bolt/_lym/config/gate.yaml` has `batch: {tiers: [0, 1], scope: card}`, with no `reports`.
   - The `batch` addition and the operator's "writer gets the note" goal depend on how keys merge with the template.
   - Fix: the record should state how an existing hub picks the step up.

5. **LOW — D3, reuse by the redundancy step.** Today `redundancy.go` reads only the hub's shared map dir. D3 changes that read path.
   - It survives: the kept set already leaves out tests the step judges by name, and tests whose hash changed, so a fresh map holding the new tests does not let them cover themselves.
   - The report's `Drift` lines are computed against the map commit. A worktree map at HEAD has no drift, so `Drift` changes meaning; worth a line in D3.

**Paths traced**
- D1 index file in the shared map dir:
  - Map readers go one package at a time through `ReadMap`, and nothing lists the map dir, so the index file survives them.
  - Writers of the shared dir are `--map` in the prime and D3's step in the prime. The second breaks the single-writer rule (finding 3).
- D2 worktree-files diff versus the map's HEAD-only write check: breaks for a dirty tree (finding 2).
- D3 worktree map dir:
  - `lym gate redundant` off the prime reads the same dir and survives.
  - The redundancy step's kept set survives (finding 5).
  - The map write's HEAD and dirty check breaks the build on a dirty tree (finding 2).
- D3/D10 `reach.json` and `mutation.json` in the gate dir: they sit beside `redundancy.json` with no other writer, and the rubric fill reads and matches them per call. They survive.
- D5/D10 rubric fill: `redundancyReportNote` re-reads the report at each fill, and the new markers follow the same path. They survive.
- D7 failure report:
  - `FailedTests` / `FailedTestsCall` and `TierRunFailures` break if the line indent or the cap is wrong (finding 1).
  - The fixer reads the report file it is pointed at and survives.
- D12 config check: the report-name check in `gateslot/config.go` is covered by D12. The template-to-hub merge is unstated (finding 4).

**Tool calls**
1. Read `internal/gateslot/runlog.go` lines 280–400: the failure-report parsers were not in context.
2. Grepped for directory listings and `testcover.Dir` callers: needed to know who lists or writes the map dirs.
3. Read `gatecli/testmap.go` `coverageMapsDir` and grepped its dirty check: the prime routing and the dirty-tree refusal were not in context.
4. Grepped `redundancy.go` and `JudgeAgainst`: to check whether reused fresh maps let new tests cover themselves.
5. Read `redundancy.go` lines 590–612: to confirm the kept set leaves out judged names and tests with changed hashes.
