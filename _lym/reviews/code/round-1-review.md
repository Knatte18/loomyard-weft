---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: internal/fabricengine/doc.go:1130-1133
    summary: The hub workspace section of the fabricengine module doc still says settings belong in the prime's settings.json, which every workspace file splices verbatim, but a beam's file now splices the beam's own settings.
    fix: In the "The hub workspace file" section of internal/fabricengine/doc.go, state that a beam's file splices the beam's own .vscode/settings.json when it exists and the prime's otherwise, and limit the sentence "settings belong in the prime's .vscode/settings.json, which the file splices verbatim" to the hub file.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/boardengine/replay.go:57
    summary: The new beam_color replayFields row, and the beam_color drop in normalizeTask at task.go:279, have no test, though the record's acceptance criteria name load, save and replay.
    fix: Add a beam_color case to the replay table in internal/boardengine/replay_test.go, such as a replayed commit that sets beam_color on a beam project over an origin that lacks it and must keep it, using the existing asProject and asBeam helpers. Add a beam_color assertion to the existing normalizeTask or round-trip test for a non-beam entry, as a row of that test and not a new test.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/hubworkspace.go:135-142
    summary: A beam whose anchor record or settings.json cannot be read makes beamWorkspaceSettings return an error that aborts the whole regeneration after the hub file is written, so the later beams' files and the stale-file removal are skipped.
    fix: In regenerateHubWorkspace, on an error from beamWorkspaceSettings log a Warn naming the beam and fall back to the prime's settings for that beam instead of returning, so the remaining beams and removeStaleBeamWorkspaces still run. Add a row for the unreadable beam settings to the beam-settings test in hubworkspace_integration_test.go.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/boardengine/store.go:456-458
    summary: UpsertPayloadKeys and the lym board upsert help name owner_machine and beam as accepted round-trip keys but say nothing of beam_color, which lym board get now emits on a beam, so a read-edit-upsert of that payload is refused as an unknown field.
    fix: Either state in the UpsertPayloadKeys comment and the payload help at internal/boardcli/cli.go:500-508 that beam_color is output-only and set by the owner setter, or accept it in the optional list as a no-op like owner_machine. Pick one and cover it with a row in the upsert key tests.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/ideengine/spawn.go (lookFor, beamColors and parentOf error branches)
    summary: The degradation paths of lookFor, an unreadable board and a failing ParentOf, each falling back to a task under the prime with a Warn, have no test.
    fix: Add two rows to TestLookFor in internal/ideengine/look_test.go, one with an unreadable board and one where ParentOf fails, each expecting green on the title bar and a non-green activity bar.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/beamcli/create.go:164
    summary: No test asserts that lym beam create passes the colour picker, since the fake SetOwners ignores BeamColor, so dropping the field would break nothing.
    fix: In the BeamRecordedHereWithNoColourRunsTheOwnerStepForOne row of TestCreate in internal/beamcli/create_test.go, have the fake SetOwners record whether req.BeamColor is non-nil and assert it for the owner step.
  - id: F7
    severity: LOW
    class: consistency
    origin: handler
    location: internal/beamcli/doc.go:22
    summary: Step 2 of the create steps in the beamcli doc says only that the owner step records the owner, beam flag and recipe, but the step now also records the beam's colour and re-runs for a beam recorded here with no colour.
    fix: In internal/beamcli/doc.go step 2, add that SetOwners also records the beam's colour when it has none, and that a beam recorded here with no colour runs the step again.
---

### [MEDIUM] fabricengine module doc still describes the prime-only settings splice

**Location:** internal/fabricengine/doc.go:1130-1133
**Issue:** `beamWorkspaceSettings` in `hubworkspace.go` makes a beam's `.code-workspace` splice the beam's own `.vscode/settings.json` (D8), and the function's doc comment says so. The module doc's "The hub workspace file" section still says settings belong in the prime's settings.json, "which the file splices verbatim". That is false for a beam file. The record's scope lists the touched package docs, and CLAUDE.md requires docs in the same commit. `internal/fabricengine/doc.go` is not in the diff.

### [MEDIUM] beam_color replay and normalizeTask paths are untested

**Location:** internal/boardengine/replay.go:57
**Issue:** The record's last acceptance criterion is that the board keeps `beam_color` through load, save and replay and drops it from a non-beam entry. Rows exist for `migrateEntries`, `ownersOf` and `SetOwners`. No replay test references `BeamColor`, so removing the `replayFields` row would pass every test and lose a colour a concurrent replayed write recorded. `normalizeTask`'s drop at `task.go:279` is also unasserted, since `migrate_test.go` covers only `migrateEntries`.

### [LOW] One unreadable beam aborts the whole workspace regeneration

**Location:** internal/fabricengine/hubworkspace.go:135-142
**Issue:** `BuildWorkspace` splices settings verbatim and never rejects them, so the only failures are an unreadable anchor record or settings file. Such a failure returns from inside the beam loop after the hub file is written. The remaining beam files and `removeStaleBeamWorkspaces` are skipped, and the caller only logs a Warn. Before this change nothing per beam could fail this way. A beam that cannot supply its own settings should fall back to the prime's.

### [LOW] beam_color is neither an upsert key nor documented as output-only

**Location:** internal/boardengine/store.go:456-458
**Issue:** D1 makes `beam_color` no upsert key on purpose. `owner_machine` and `beam` are listed so a payload from `lym board get` round-trips, and `beam_color` now appears in that payload for a beam. The upsert validator rejects it as unknown, and neither the `UpsertPayloadKeys` comment nor the payload help says it is output-only.

### [LOW] lookFor degradation paths have no test

**Location:** internal/ideengine/spawn.go (lookFor)
**Issue:** `lookFor` documents that an unreadable board or an unresolvable parent reads the worktree as a task under the prime with a Warn. `look_test.go` covers prime, beam, unrecorded beam, task under a beam, siblings, a done beam and the missing shortname, but neither fallback.

### [LOW] Nothing pins that beam create passes the colour picker

**Location:** internal/beamcli/create.go:164
**Issue:** `createDeps` fakes `SetOwners`, and the fake ignores `BeamColor`. The new `TestCreate` row asserts only the call list, so a request without the picker would pass.

### [LOW] beamcli module doc omits the colour step

**Location:** internal/beamcli/doc.go:22
**Issue:** Step 2 of the create steps does not mention that the owner write records the beam's colour, nor that a beam recorded here without one runs the step again (D2).

## Rejected

- Lens code-correctness, blank or comment-only settings.json left untouched: D4 says a file that does not decode as JSONC is left untouched with a warning, and the code does that.
- Lens code-correctness, colour read for an unrecorded beam can differ from the one later recorded: D3 says the read colour is picked "the same way" as `lym beam create` would, and a one-time change at recording is inherent to a colour not yet recorded.
- Lens code-tests, `beamWorkspaceSettings` failure-path test: dropped as a separate finding; F3 asks for its row with the fix.
- Lens code-tests, `normalizeTask` as its own LOW: merged into F2.
- Lens code-tests NIT on overlap between `look_test.go` and `color_test.go`: the lens itself judged the rows non-redundant, since `look_test.go` pins the wiring from `lookFor`.
- `internal/battenshed/template.yaml` in the diff without a scope bullet: a comment-only change that keeps the `seed_ide_command` description true to what `lym ide seed` now writes, matching the `docs/overview.md` edit. The board entry licenses it by changing what seed writes.
- Lens code-pattern-fit and the handler found no violation of `PATTERN-pair-parent`, `PATTERN-owner-machine`, `PATTERN-tolerant-records` or `PATTERN-leaf-packages`; no declared PATTERN change exists and no commit carries a `Pattern-Departure:` trailer.
