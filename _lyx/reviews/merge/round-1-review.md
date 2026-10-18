---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/configreg/runsettings_test.go:326
    summary: No test pins that a run profile naming a batten command key is refused by the local layer's executable category
    fix: 'In TestLayerRunSettings in internal/configreg/runsettings_test.go, add a row whose profile sets open_ide_command under the batten module to the string x and whose task settings are empty, expecting a *RunSettingsError for module batten that names open_ide_command and wraps the profile name.'
    origin: lens:code-tests
---

### [LOW] No test for a profile holding an executable batten key

**Location:** internal/configreg/runsettings_test.go:326
**Issue:** The task made `open_window_command`, `seed_ide_command` and `open_ide_command` `CategoryExecutable`, so the local layer refuses them.
The parent's `LayerRunSettings` reaches the same refusal for a profile through `PrepareRunSettings(profile)` (`internal/configreg/runsettings.go`, `checkLocalDocument`).
The code is correct: the merged behavior was traced and a profile with such a key stands Seed-Child Stuck.
Only the task-side refusal (`open_ide_command` in a task's `run_settings`) and the registry classification (`wantLocalDenied`) have tests, and the existing profile-side refusal row uses a non-executable key.
The interaction of a profile with a Command row's command key has no coverage.

## Rejected

- Lens code-tests NIT, "a task switch over a profile switch has no batten-key row": the rows of F1 can carry it, and the layering behavior is already pinned with generic keys.
- Focus items on `wire.go` signatures, `envkit.FullEnv` and test doubles: every closure, double and stub in the merged tree uses `battenshed.RunSettings`, and `wire.go` reaches `prepareRunSettings`.
- Focus item on `PATTERN-config-strictness`: `battenshed` still uses `LoadOrTemplate`, the task's executable-key lookup goes through strict `configreg.SettingLookup`, and `runprofile` is the parent's declared exception.
- Focus item on `lock`: the task adds `TryAcquireReadLock`, and the parent removed `LockGrace` and widened a different row of `TestAcquireWriteLockWithin`; no overlap in contract or timing.
- Focus item on the reed probe socket, creel mail and creelengine renewals: no caller the task rewired depends on them.
- Focus item on docs: `battenshed/doc.go` carries both the profile text and the Command-row text without contradiction, and no stale "switched row" wording remains.
