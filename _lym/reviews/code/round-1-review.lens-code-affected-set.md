```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff's changes force outside the lines it touches.

**Per change, the set found**

- **Dobby's per-tick landing step (`landNotice` → `landBeam`):**
  - The only caller is `tick` at `internal/dobbyengine/loop.go`.
  - Docs: the package doc `internal/dobbyengine/doc.go` and `internal/beamcli/doc.go` are updated.
  - Tests: `land_test.go` replaces `landnotice_test.go`.
  - State key: the `land_notice` → `land` rename has no other reader or fixture.
  - `loop.go:526` is not covered. See F1.
- **`beam ready` verb:**
  - Registered in `Command()`.
  - Doc coverage: the `cli.go` header and `Long` text, the `beamcli/doc.go` section and verb list, and `docs/overview.md:449`.
  - `cmd/lym` tests: `shortcuts_test.go` and `removedverbs_test.go` do not enumerate beam verbs, so nothing is missing there.
  - The orch stencils and `plugins/` have no `beam land` or can-land text. Covered.
- **`lym beam land` unflagged refusal:**
  - Doc: `landCmd` `Long` and `beamcli/doc.go` list it.
  - Test: the `TestLand` row exists.
  - Callers and doc references of `judgeLandEntry` and `onHubLandDeps` are all updated, and `landFromHub` has no remaining references.
  - Covered.
- **Beam-Await re-check (`pawlRecheck`, `pawlRecheckLines`):**
  - The one non-test caller and the test are updated.
  - Not covered: `internal/loomcli/doc.go:14` still lists only blockers, refusal and owners. See F2.
  - `pawlengine/doc.go` and `pawlengine/beamawait.go:318` ("fix them, then run `lym beam land`") remain accurate with the new line.
- **Ready record paths (`BeamReadyPath`, `BeamReadyLockPath`):**
  - They follow `LandingDirRel` and `lymdirs`, mirroring `_lym/landing` and `.lym/landing`.
  - Other readers and writers of the landing directory:
    - `fabricengine.CommitAnchoredPaths` is called with `LandingDirRel()`, so the record is committed.
    - Nothing else enumerates files in that directory, so the new file does not disturb the change-description readers.
  - Covered.
- **`pawlengine` doc and `docs/overview.md:271` (the `pawlengine` package line):** both updated.
- **PATTERN entries:**
  - `PATTERN-refusal-way-forward` is satisfied. The new refusals are `land.go:62`, the `ready` refusals, and the re-check line, each naming `lym beam ready <slug>` or a fix-and-retry step.
  - `PATTERN-tolerant-records` is satisfied. The record goes through `state.WriteJSON` and `state.ReadJSON`.
  - `PATTERN-fabric-git` is satisfied. The commit goes through `fabricengine`.
  - `PATTERN-durable-vs-ephemeral-state` is satisfied. The paths are named through `loomengine` and `lymdirs`.
  - `PATTERN-dobby-starts-runs` and `PATTERN-owner-machine`: dobby's `Land` calls `land()`, which keeps the owner checks. No violation found.
  - No `Pattern-Departure:` trailer is present, and no declared PATTERN changes appear in the directive, so there is nothing to check there.
- **Commit message and summary:**
  - The commit message's claim that dobby posts "only when it cannot" matches `land.go`.

**Findings**

- **F1, LOW, consistency:** `internal/dobbyengine/loop.go:526` — the `tick` doc comment still ends "ending with the beam's "can land" notice", but the last step is now `landBeam`, which lands the beam and posts a notice only when it cannot. Evidence: `loop.go:526`, with the call at `loop.go:576`. Fix: reword the ending to "ending with the beam's landing step".
- **F2, LOW, consistency:** `internal/loomcli/doc.go:14` — the Beam-Await re-check paragraph lists the blockers, the refusal and the two owners. It omits the ready record, which `pawlRecheck` now reads and `pawlRecheckLines` words as `lym beam ready <slug>`. The decision record scoped the `loomcli` package doc for this change. Evidence: `loomcli/doc.go:14`, `pawlwiring.go:230-254`. Fix: add that the re-check also reads the beam's ready record and names `lym beam ready <beam>` for an unflagged beam.

**Tool calls (3)**
1. A repo-wide grep for stale can-land and land-notice wording, plus `loop.go:526` and the `docs/overview.md` beam lines. My context did not hold the call sites outside the diff's files.
2. A list of files naming beam start/stop, the callers of the changed functions, and the `loomengine` path declarations. This checked verb-enumerating files and other users of the landing directory.
3. A grep of `contracts/stencils/orch` and `plugins` for beam land and can-land wording, the `cmd/lym` verb-list tests, and `loomcli/doc.go` lines 10-18. This checked the stencils for stale wording and read the `loomcli` package doc.
