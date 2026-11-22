---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: "internal/dobbyengine/reinterpret.go:37"
    summary: "The required-to-on rewrite loses requirements while claiming to preserve their meaning."
    fix: "Flag the non-equivalent conversion and give an actionable migration choice for the lost mandatory-PR and origin requirements."
    origin: "lens:code-correctness"
  - id: F2
    severity: BLOCKING
    class: design
    location: "internal/hubgeom/tasksetting.go:32"
    summary: "The chain resolves other worktrees through an API forbidden by the cwd-resolution pattern."
    fix: "Obtain each other-worktree location through lymcwd.NamedWorktree(hubPath, name), retaining the deliberate existence check."
    origin: "lens:code-affected-set"
  - id: F3
    severity: BLOCKING
    class: consistency
    location: "internal/boardengine/board_test.go:574"
    summary: "Three retained contained-test candidates lack the required keep markers for their distinct assertions."
    fix: "Add //testtiming:keep above TestEntryDates, TestLookupTaskSettings_RefusesAValueTheKeyDoesNotTake, and TestPRReview_NoStopsTheRecordedSessionBeforeDroppingTheRecord, naming each distinct assertion."
    origin: "lens:code-tests"
  - id: F4
    severity: MEDIUM
    class: design
    location: "internal/dobbyengine/tasksetting.go:61"
    summary: "The new lookup rejects environment markers accepted by the existing config loader."
    fix: "Resolve the selected own-file value through the existing config expansion policy before validation, preserving provenance and avoiding merged defaults."
    origin: "lens:code-correctness"
  - id: F5
    severity: MEDIUM
    class: design
    location: "internal/boardengine/store.go:275"
    summary: "Migrated board overrides never reach already-seeded runs."
    fix: "Materialize migrated settings once in existing runs' effective local layers before switching their consumers, preserving explicit new values."
    origin: "lens:code-correctness"
  - id: F6
    severity: MEDIUM
    class: design
    location: "internal/boardengine/store.go:269"
    summary: "The migration discards meaningful project landing-notice requests."
    fix: "Preserve legacy project notice requests through an effective migration path before deleting the old field, without overwriting explicit new settings."
    origin: "lens:code-correctness"
  - id: F7
    severity: MEDIUM
    class: design
    location: "internal/boardengine/store.go:311"
    summary: "Committed-board replay compares unconverted historical entries against converted current entries."
    fix: "Apply the same legacy-field normalization to historical boards parsed for replay before comparing entries; cover a legacy notice edit and removal."
    origin: "lens:code-affected-set"
  - id: F8
    severity: MEDIUM
    class: scope
    location: "internal/dobbyengine/landnotice.go:123"
    summary: "A recorded beam PR overrides an explicit off setting without an approved behavior change."
    fix: "Honor the resolved off value for beam members and restore the suppression assertion; retain PR follow-up for enabled notices."
    origin: "lens:code-scope"
  - id: F9
    severity: MEDIUM
    class: design
    location: "internal/hubgeom/tasksetting_integration_test.go:52"
    summary: "The chain reader's cycle and persisted-state failure contracts lack coverage."
    fix: "Add rows to the existing chain test for cycle termination and malformed parent or anchor state, asserting the chain or propagated error."
    origin: "lens:code-tests"
  - id: F10
    severity: LOW
    class: consistency
    location: "plugins/ly/skills/board/SKILL.md:260"
    summary: "The board skill still promises the removed notice-based prune exception."
    fix: "Replace the obsolete notify_on_land retention instruction with the current prune and pending-notice contract."
    origin: "lens:code-affected-set"
---

### [BLOCKING] Required-mode migration silently weakens landing requirements

**Location:** internal/dobbyengine/reinterpret.go:37

**Issue:** Rewriting `required` to `on` is not equivalent. Previously, a non-empty beam without a recorded PR required one; the new `on` resolver wants none. An ordinary task lacking usable origin facts can also change from a halt to a local landing. The rewrite reports that meaning was retained. `PATTERN-config-layers` requires preserving the old meaning or flagging it with a way forward. No departure authorizes this conversion.

### [BLOCKING] Other-worktree resolution bypasses the prescribed API

**Location:** internal/hubgeom/tasksetting.go:32

**Issue:** The chain calls `lymcwd.ResolveWorktree(root)` for task and parent worktrees. `pattern/PATTERN-cwd-resolution.md:15` explicitly requires callers outside `lymcwd` to obtain another worktree's location through `lymcwd.NamedWorktree(hubPath, name)`. That API documents precisely this role. The separate existence check does not waive the boundary.

### [BLOCKING] Distinct contained tests need keep annotations

**Location:** internal/boardengine/board_test.go:574

**Issue:** Three supplied contained candidates have distinct assertions and should remain, but lack the annotations required by `PATTERN-test-speed`. `TestEntryDates` asserts exact creation and revision timestamps across writes. `TestLookupTaskSettings_RefusesAValueTheKeyDoesNotTake` at `internal/dobbyengine/tasksetting_test.go:135` additionally asserts the offending file path. `TestPRReview_NoStopsTheRecordedSessionBeforeDroppingTheRecord` at `internal/landingshed/prreview_session_test.go:502` asserts the stopped strand identity; its named covering row has no recorded strand and does not assert stops. Their named covering tests do not subsume these assertions.

### [MEDIUM] Valid environment-backed settings fail the new lookup

**Location:** internal/dobbyengine/tasksetting.go:61

**Issue:** A value such as `pull_request: "${env:PR_MODE:-off}"` is accepted by `LoadConfig`, which builds the expansion environment and resolves markers. `TopLevelValue` instead returns raw YAML, and the new lookup immediately validates that marker as a literal mode. PR-Review and Finalize therefore halt on valid existing configuration. The same mismatch affects `land_notice`. Selecting the winning own-file value must preserve the loader's expansion semantics.

### [MEDIUM] Existing runs lose migrated overrides

**Location:** internal/boardengine/store.go:275

**Issue:** The fold transfers legacy values only into board `RunSettings`, while new consumers read worktree files. Those settings reach local files during Seed-Child; resuming an existing run does not repeat that step. An active task previously carrying `pull_request: off` or `merge-in` therefore inherits another mode after upgrade. A legacy notice request can likewise become the default `off`. The board CLI explicitly says later run-setting changes apply to the next run, and the landing summary acknowledges this gap. The required carry must preserve existing runs' effective overrides.

### [MEDIUM] Project notice requests are deleted without a replacement

**Location:** internal/boardengine/store.go:269

**Issue:** The fold deletes both legacy keys and then returns for projects. Although the former project PR field was ignored, project `notify_on_land: true` was meaningful: the old notice reader accepted project requests, and done-entry handling had no kind restriction. A beam project with that request and default notice setting `off` loses it on save. The comment claiming that no run read these project keys is therefore false for notices.

### [MEDIUM] Replay omits the legacy-field migration

**Location:** internal/boardengine/store.go:311

**Issue:** `Store.Load` folds legacy fields, but `ParseBoardTasks`, used by `parseReplayBoard`, does not. Replay consequently compares unconverted base, parent and commit entries against a converted origin. A pre-upgrade commit changing only `notify_on_land` changes only `Unknown`; the new replay field table sees no change and silently returns. An otherwise unchanged removal carrying a legacy PR value can also falsely conflict because only origin has the replacement `RunSettings`. All comparison inputs need the same normalization.

### [MEDIUM] Beam membership defeats an explicit notice-off value

**Location:** internal/dobbyengine/landnotice.go:123

**Issue:** The new condition sends a notice for a member of a beam with a recorded PR even when lookup returned false. Previously, `off` suppressed it. The decision replaces the source and modes and requires lower settings to override in both directions; it does not authorize this exception. The landing summary's acknowledgement is not an approved amendment. The test change also reverses `OffWithAPullRequestFlagsTheMemberAndSendsNoLine` into an expectation of delivery, without a `Loosened-Check:` trailer in the reviewed commits.

### [MEDIUM] Chain failure behavior lacks contract tests

**Location:** internal/hubgeom/tasksetting_integration_test.go:52

**Issue:** The new test covers an ordinary task beneath a beam, an absent worktree and the prime. The implementation also promises cycle termination and error propagation for failures resolving a present worktree or reading its parent. These persisted-state boundaries determine whether a landing halts or inherits another setting, but no rows exercise them.

### [LOW] The board skill describes obsolete prune behavior

**Location:** plugins/ly/skills/board/SKILL.md:260

**Issue:** The current skill says prune retains an entry carrying `notify_on_land` until dobby sends and clears it. The changed prune implementation removes done entries without that exception, and the field no longer exists. Agents following this instruction will rely on retention the implementation no longer provides.

