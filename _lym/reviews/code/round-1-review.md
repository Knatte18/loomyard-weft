---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/dobbyengine/land.go:57
    summary: a transient Land refusal, such as the run not yet at Beam-Await, posts a notice and records the episode, so dobby never retries and a flagged beam does not land
    fix: In landBeam, when d.deps.Land returns an error, post the refusal notice once per episode but do not call recordLand for a refusal that only waits on run state; let the next tick call Land again. Keep the episode recorded only after Land succeeds or the parent-owner notice goes out. One way is to track a separate notified marker in LandState so a retried refusal posts no second notice. Add a TestLandBeam row where Land refuses once, then succeeds, and assert one notice and one landing.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/beamcli/dobbydeps.go:172
    summary: no test pins beamLandReading returning not passing for an unflagged beam before any other read, nor that deps.Land runs the land sequence and returns its refusals
    fix: Add a tier0 test in internal/beamcli that builds the dobby deps over a fake or hermetic hub (hubforge) and asserts that beamLandReading for an unflagged beam returns Passes false without calling ReadBeamLandBlockers, and that deps.Land for an unflagged beam returns the land refusal naming `lym beam ready <slug>` and for a flagged beam writes the same land request `lym beam land` writes.
  - id: F3
    severity: LOW
    class: design
    origin: handler
    location: internal/beamcli/deps.go:125
    summary: MarkReady writes the record before the commit, so a failed commit leaves an uncommitted flag and the way forward to run the verb again reports already without committing
    fix: In the MarkReady dep, make the verb idempotent for the commit, so that a record already present but uncommitted is committed by CommitAnchoredPaths on the next run; for example have markReady call MarkReady's commit step for an already flagged beam too, since CommitAnchoredPaths reports committed false for a clean path. Alternatively remove the written record when the commit fails. Add a TestMarkReady row for a failing MarkReady dep.
  - id: F4
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/beamcli/land_test.go:186
    summary: the ready refusal's position after the owner checks and before the work-remaining refusal, and the Ready and MarkReady error ways forward, have no test row
    fix: In TestLand add a row with an owner refusal and an unflagged beam expecting the owner refusal, and a row with w.refusal set and an unflagged beam expecting the ready refusal with refusedFor empty. Give landWorld an error field for Ready and MarkReady and add rows asserting each error's way forward text in TestLand and TestMarkReady.
  - id: F5
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/loomcli/pawlwiring.go:230
    summary: pawlRecheck's new ReadyFor read of the beam's own record has no test, and the order of the unflagged line is not pinned by the every-failure row
    fix: Set unflagged on the EveryFailureGivesItsOwnLineInOrder row of TestPawlRecheckLines and add the ready line last in its want. If pawlRecheck can be driven over a hermetic worktree, add a case that reads the beam's own record; otherwise leave the read covered by F2's beam-level test.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/dobbyengine/loop.go:526
    summary: the tick doc comment still ends with the beam's can land notice, though the last step is now the landing step
    fix: In the doc comment of tick, replace "ending with the beam's "can land" notice" with "ending with the beam's landing step".
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/doc.go:14
    summary: the package doc's Beam-Await re-check paragraph omits the ready record that pawlRecheck now reads
    fix: In the Beam-Await re-check paragraph of internal/loomcli/doc.go, add that the re-check also reads the beam's ready record and words an unflagged beam as `lym beam ready <slug>`.
---

### [MEDIUM] A transient Land refusal ends the episode for good

**Location:** internal/dobbyengine/land.go:57
**Issue:** `landBeam` sends a `Land` refusal to `postLandNotice`, which records the episode. `Land` runs the full `land` sequence, whose `judgeLandRun` refuses while the pawl run is not waiting at Beam-Await. A beam flagged right after `lym beam create` can hit that window. The reading keeps passing, so the dobby never calls `Land` again once the run reaches Beam-Await, and the beam stays flagged and unlanded until someone runs `lym beam land`. The amended entry says dobby lands a flagged beam itself as soon as the flag is set. The decision record's "not retried within the same episode" covers a hard refusal, not this wait.

### [MEDIUM] The beam-level seams of the landing gate are untested

**Location:** internal/beamcli/dobbydeps.go:172
**Issue:** The `TestLandBeam` rows drive fake `CanLand` and `Land` seams. Nothing exercises `beamLandReading`'s ready check, which is the seam that stops dobby from landing an empty new beam, so removing it fails no test. Nothing shows that `deps.Land` writes the same land request as `lym beam land` and returns its refusals.

### [LOW] A failed commit leaves an uncommitted flag that a rerun reports as already set

**Location:** internal/beamcli/deps.go:125
**Issue:** `MarkReady` writes the record and then commits. If the commit fails, the verb tells the caller to run it again, but the rerun reads `Ready` as true, reports `already` and never commits. The flag works locally but is not tracked, so it does not survive a machine change, which the design relies on.

### [LOW] Ready refusal ordering and error paths are unpinned

**Location:** internal/beamcli/land_test.go:186
**Issue:** The unflagged row leaves the owner checks and the work-remaining refusal passing, so moving the ready check anywhere still passes. The `Ready` and `MarkReady` error ways forward are never reached, and `PATTERN-refusal-way-forward` asks for a test that reaches each new refusal.

### [LOW] pawlRecheck's ready read and the line order are unpinned

**Location:** internal/loomcli/pawlwiring.go:230
**Issue:** `TestPawlRecheckLines` covers only the pure line builder. The every-failure row does not set `unflagged`, so the position of the new line is not pinned.

### [LOW] Stale tick comment names the can-land notice

**Location:** internal/dobbyengine/loop.go:526
**Issue:** The comment says the tick ends with the beam's "can land" notice, though `landBeam` now lands the beam and posts only when it cannot.

### [LOW] loomcli package doc omits the ready record in the re-check

**Location:** internal/loomcli/doc.go:14
**Issue:** The Beam-Await re-check paragraph lists blockers, refusal and owners and not the ready record. The decision record scoped this package doc for the change.

## Rejected

- Lens code-tests 6: a test that `BeamReadyPath` and `BeamReadyLockPath` mirror `_lym/landing` and `.lym/landing`. The functions are one-line joins over `LandingDir` and `lymdirs`, and the other lenses found the paths correct; a path test would restate the code.
- Lens code-tests 7: unasserted `record` and `way_forward` result keys and the corrupt-record read error. These pin incidental output, not a behavior the entry asks for.
- Lens code-tests 8: a failed state write after a successful `Land`. The next tick rewrites the same request while Beam-Await has not consumed it, which the correctness lens found harmless.
- Lens code-correctness: the `land_notice` to `land` state key rename. An older state file is read tolerantly and the old key is ignored, so no finding.

All named focus questions were settled with no departure: the unflagged refusal sits after the owner checks and before the work-remaining refusal and names `lym beam ready <slug>` then `lym beam land <slug>`; the record goes through `internal/state` with paths mirrored through `loomengine`; the re-check adds the unflagged line with the beam's own slug; and the docs of beamcli, dobbyengine, pawlengine and `docs/overview.md` match the new behavior, bar the loomcli doc in F7.
