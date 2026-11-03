---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lym/landing/summary.md:27
    summary: The summary says none of the beam's code that main's commits read changed under it, but the beam extended fabricengine.ConfigDirs with a Records field and main's hibernatecli reads it
    fix: In _lym/landing/summary.md, reword the closing sentence of "How the two sides met" ("Main changed no stencil the beam moved into an engine block, and none of the beam's code that main's commits read changed under it.") so it names the one exception. Keep the stencil claim as it is. Replace the second half with "the one beam change main's commits read is `fabricengine.ConfigDirs`, which now also tells the weft checkout in a `Records` field that no config layer reads, so `internal/hibernatecli`'s `hibernateconfig.LoadConfig` call is unaffected."
    origin: handler
---

### [LOW] The summary's "nothing the beam changed under main's reads" claim is not borne out for `ConfigDirs`

**Location:** `_lym/landing/summary.md:27`, against `internal/fabricengine/junctionnames.go` (`ConfigDirs`), `internal/configengine/layers.go` (`Dirs`) and `internal/hibernatecli/cli.go:101`.

**Issue:** The summary closes with "none of the beam's code that main's commits read changed under it".
The beam's commits changed `fabricengine.ConfigDirs` to return a third field, `Records`, and added that field to `configengine.Dirs`.
Main's commit `68f39997` calls `fabricengine.ConfigDirs(location)` in `chooseResumeOnWake` and hands the result to `hibernateconfig.LoadConfig`, so the claim is false as written.
I checked the interaction and it is harmless: `configengine` reads `Dirs.Anchor` and `Dirs.Board` only, so the hibernate load resolves exactly as it did on main, and `go build` and `go vet` pass on every package in the range.
The defect is the summary sentence alone.

## What I checked and found sound

- The range is the net import of main's nine commits, and no more.
  The file set of `c18e033a..HEAD` equals the file set of main's `7f0f65b3..f5724733`.
  For each file, the added and removed lines of the range diff equal those of main's own patch.
  The merge added no edit that neither side carries, so the "no new behaviour" bound holds.
- `internal/loomcli/wiring.go` keeps both sides.
  The range diff over the file holds only main's pre-push hunks, in `loomCommitStatusDeps`, `commitStatusDepsGuardedBy` and `newCommitStatusSeam`.
  The beam's `DefaultEngine` fields, `assistBounds`, `segmentAssistBounds`, `assistBound` and the `Assist` settings in `loomSegmentSettings` are all present, unshadowed and still wired in `wire()`.
- `configreg_test.go`, `cmd/lym/configstrictness_test.go`, `docs/overview.md` and `PATTERN.md` each carry main's change beside the beam's.
  `PATTERN-engine-blocks` and `PATTERN-fs-links` are still in `PATTERN.md`, and main's `PATTERN-owner-machine` and `PATTERN-config-layers` changes read consistently with the code.
- The one type change main made across a seam is the Waiting hook's `WaitNote` return and `PrintStatusLinesOnChange`'s `(line, key)` poll.
  The beam adds no other implementer or caller of either, and the `arm.go` and `checklist.go` callers compile against the new signatures.
- Main's other seams against the beam's work compile and behave: `fabricengine` leftover removal, batten's `ChildRecipeFile`, beam `ClearMotifRun`, dobby `drivable`, and the `hibernate` config row.
  None reads a beam-changed signature, default or path other than `ConfigDirs` above.
- No commit in the range carries a `Pattern-Departure:` trailer, and the focus directive declares no PATTERN changes, so there is nothing to check under either rubric bullet.
- Merge-time test edits are one-row additions (`hibernate` rows) that duplicate no coverage, and each behaviour main added brought its own tests in main's commits.

## Rejected

- The lens reports raised no findings, so none was kept or rejected.
