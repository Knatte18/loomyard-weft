```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens review of a docs-only diff (README index line, three proposals, one report).

**Per change, the set found**

1. `docs/research/README.md` index line (new entry for `testmap-upkeep-2026-11-11`).
   - Set: the neighbouring entries in the `speed` section, the README's own format rules (lines 1–8), and the target folder.
   - The line follows the neighbours' format: a link to `speed/<name>/report.md`, an em dash, then a lowercase description ending in a full stop.
   - The target folder holds `report.md` beside the proposals, which the README allows ("a folder that holds `report.md` beside the scripts and data").
   - Covered. No finding.

2. New study folder `docs/research/speed/testmap-upkeep-2026-11-11/` with `report.md` and three proposals.
   - Set: the report's relative links to `proposal-fable-a.md`, `proposal-fable-b.md` and `proposal-opus.md`, and the README link.
   - All three proposal files exist in the folder at those names, so the links resolve. Covered. No finding.
   - The repo constraint documents (`CLAUDE.md`, `PATTERN.md`, `pattern/*.md`) have no rule on where research reports live or how they are named. The search for `docs/research` in those files returned nothing. Nothing is forced.
   - The folder name follows the neighbours' `<topic>-<date>` pattern. Covered.

3. Code identifiers and behaviours cited in the report and proposals.
   - Set: `internal/testcover`, `internal/gatecli/testmap.go`, `testmappackage.go`, `internal/dobbyengine/testmap.go`, `internal/gateslot`, and the `test_map_*` config defaults.
   - Verified in the code at HEAD:
     - `test_map_every: 5` and `test_map_batch: 5` in `internal/dobbyengine/template.yaml`.
     - `test_map_max_age_days: 7` in `internal/gateslot/template.yaml`.
     - `slots: 2` in the template, and `slots: 3` in the hub's `gate.yaml`.
     - `testMapStep` and `LandingsSinceTestMap` in `internal/dobbyengine`.
     - `ExitRecord.Unbuilt` written at `internal/gatecli/testmap.go:482` and read by no production code.
     - `writeFailed` in `testmappackage.go`.
     - `Pool.tryAcquireAny` unexported, with `Holders` and `Draining` exported.
     - `ScanSpawns` called only from `internal/gatecli/redundancy.go`.
     - `judgeRuns` in `redundancy.go`.
     - 4424 tier 0 and 903 tier 1 `tiergate.Require` call sites.
   - Every cited identifier exists and behaves as the report says. Covered. No finding.

4. The report's "Claims checked" row "First-parent commits … 173 | O | 172 by `git log`".
   - My count over the same window on `main` was 171, and it is sensitive to the date bounds.
   - This is a snapshot figure in a measurement report, which the prose rules exempt. Not a finding.

5. No PATTERN or `pattern/` change is declared for this diff, so "Declared PATTERN changes landed" raises nothing. The only commit carries no `Pattern-Departure:` trailer and needs none, because the diff touches no code. No stencil is changed.

**Findings:** none.

**Tool calls (3):**
1. `sed`/`ls`/`grep` over `docs/research/README.md`, the study folder, and `CLAUDE.md`/`PATTERN.md`/`pattern/*.md`, plus config and code greps. My context lacked the README format rules, the neighbour entries, and the cited code.
2. `grep` and `sed` over `internal/dobbyengine/testmap.go` and the `Unbuilt` readers. My context lacked the dobby refresh step and the `Unbuilt` call sites.
3. `grep` counts of `tiergate.Require` call sites and the first-parent commit count. My context lacked those figures.
