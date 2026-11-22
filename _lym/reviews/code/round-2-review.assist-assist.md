---
verdict: BLOCKING
findings:
  - id: F26
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/dobbyengine/reinterpret.go:159
    summary: The migration guesses every worktree's anchor from the prime and can carry settings where the runtime lookup never reads them.
    fix: Pass resolved code-worktree paths and their own anchors into the reinterpretation through the reconciliation boundary. Reuse the resolved worktree listing already obtained by hubreconcile, or resolve locations in the geometry layer through lymcwd.NamedWorktree. Remove settingPlaces' prime-offset inference and sibling directory discovery from dobbyengine. Flag and carry using each supplied anchor, and cover a prime and task with different recorded anchors.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    carries: F2
    location: internal/dobbyengine/config.go:73
    summary: The beam module name is still repeated independently of beamsetting.Module.
    fix: Replace the beam module-name literals in dobbyengine.ConfigSpec, the configreg.Modules beam registration, and beamcli.mergeParent's run_settings map with beamsetting.Module. Keep independent legacy schema field names separate.
  - id: F27
    severity: BLOCKING
    class: design
    origin: lens:code-tests
    location: internal/configreg/configreg_test.go:27
    summary: The contained registration test retains distinct assertions without the keep marker PATTERN-test-speed requires.
    fix: Add a testtiming keep directive immediately above TestLookup_BeamCarriesThePerTaskSettingsReinterpretation. State that it pins the reinterpretation ID, nonnil Rewrite and RunsWithoutFile, which TestPrepareRunSettings does not assert. Keep the test.
  - id: F28
    severity: BLOCKING
    class: design
    origin: lens:code-tests
    location: internal/configreg/runsettings_test.go:77
    summary: The added valid-beam row duplicates the existing beam local-document row in TestPrepareRunSettings.
    fix: Delete the row named a beam pull_request merge-in and a land_notice on pass. Keep the row named a task's beam pull_request and land_notice prepare into a beam local document, which has identical settings, expected output and assertions.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    carries: F5
    location: internal/boardengine/store.go:364
    summary: A board save before reconciliation removes the raw keys needed to migrate an already-seeded task's settings.
    fix: Make the one-time live-worktree migration recover folded task beam settings from run_settings as well as retired raw keys, preserving a worktree's explicit settings. Alternatively preserve durable pending migration data until the carry completes. Add a regression that loads and saves an unrelated board edit before reconciliation and verifies the running task receives its merge-in and landing-notice settings.
  - id: F12
    severity: MEDIUM
    class: decision
    origin: lens:code-scope
    carries: F12
    location: internal/dobbyengine/landnotice.go:165
    summary: A recorded beam pull request overrides a member's land_notice off without a recorded exception to the precedence rule.
    fix: Leave the code in place and defer this finding to the run's parent. The parent must record the exception with lym loom decision add or refuse it. If refused, require the resolved land_notice to be on before sending the member's line, retaining its follow-up when a line is sent.
  - id: F13
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    carries: F13
    location: internal/boardengine/store.go:298
    summary: A beam project's legacy notice is still discarded permanently if the board is saved before the worktree carry.
    fix: Preserve a beam project's pending legacy notice in durable migration data until reconciliation carries or explicitly flags it. Do not delete its only persisted representation on an unrelated board save. Cover a project with notify_on_land true, an intervening board save, and then reconciliation into the beam worktree.
prior:
  - id: F1
    status: resolved
  - id: F2
    status: open
  - id: F3
    status: resolved
  - id: F4
    status: resolved
  - id: F5
    status: open
  - id: F6
    status: resolved
    answer: The walk now flags sibling worktrees and the changed landing meaning. Its new anchor inference defect is F26.
  - id: F7
    status: resolved
  - id: F8
    status: resolved
  - id: F9
    status: resolved
  - id: F10
    status: resolved
  - id: F11
    status: resolved
    answer: Registration is asserted. The new test's missing keep marker is F27.
  - id: F12
    status: open
    answer: No listed added decision settles the exception held by the parent.
  - id: F13
    status: open
  - id: F14
    status: resolved
  - id: F15
    status: resolved
  - id: F16
    status: resolved
  - id: F17
    status: resolved
  - id: F18
    status: resolved
  - id: F19
    status: resolved
  - id: F20
    status: resolved
  - id: F21
    status: resolved
  - id: F22
    status: resolved
  - id: F23
    status: resolved
  - id: F24
    status: resolved
  - id: F25
    status: resolved
---

### [BLOCKING] Migration uses the wrong worktree anchor

**Location:** internal/dobbyengine/reinterpret.go:159

**Issue:** `settingPlaces` joins the prime's anchor offset onto every sibling worktree.
For a prime anchored at `backend` and a task anchored at `frontend`, it reads and writes `task/backend`.
`TaskSettingChain` instead reads `task/frontend` through `lymcwd.NamedWorktree`.
The migration can report a successful carry that the landing lookup never sees.
It can also leave the actual retired value unflagged.
This violates `PATTERN-cwd-resolution`'s own-anchor and told-geometry rules.
The reconciliation walk already obtains resolved anchors through `fabricengine.CodeWorktrees`.
The correctness and affected-set lenses independently found this defect; it is recorded once.

### [BLOCKING] Shared module naming remains incomplete

**Location:** internal/dobbyengine/config.go:73

**Issue:** `beamsetting.Module` declares the module name, but `ConfigSpec` repeats its literal.
The registry's changed beam row and the merge-parent task writer repeat it too.
They identify the same module that board folding now references through the constant.
F2 therefore remains open under `PATTERN-code-conventions`.

### [BLOCKING] Distinct registration assertions need their keep marker

**Location:** internal/configreg/configreg_test.go:27

**Issue:** The current redundancy report lists this test as contained by `TestPrepareRunSettings`.
Its ID, callback and absent-file assertions are distinct from that test's document and refusal assertions.
`PATTERN-test-speed` requires a kept candidate to explain that distinction with `//testtiming:keep`.
The marker is absent.
The listed-constraint severity rule makes this a BLOCKING finding.

### [BLOCKING] Duplicate beam validation row

**Location:** internal/configreg/runsettings_test.go:77

**Issue:** The added valid-beam row uses the same input and expected document as the row at line 207.
Both exercise the same assertions in `TestPrepareRunSettings`.
It adds no behavior coverage and violates `PATTERN-test-speed`'s test-economy rule.

### [MEDIUM] Folded task settings no longer reach an existing run

**Location:** internal/boardengine/store.go:364

**Issue:** `foldLegacyBeamFields` deletes the raw keys and saves their task values into `run_settings`.
`LegacyBeamSettings` reads only the retired raw keys.
An unrelated board mutation can therefore remove the migration source before the first reconciliation.
The board CLI loads its store without first invoking this reconciliation.
The later carry finds no values, while an already-seeded run never reapplies its new `run_settings`.
A merge-parent task can consequently lose `merge-in`, and a task can lose its notice request.
F5's direct migration path exists, but its ordering requirement remains unsatisfied.

### [MEDIUM] The notice exception still needs the parent's decision

**Location:** internal/dobbyengine/landnotice.go:165

**Issue:** The membership and recorded-PR branch sends a notice even when `note.Requested` is false.
The decision record requires the lower setting to override in both directions and contains no added exception.
F12 remains held by the parent; this review does not authorize reverting the exception.

### [MEDIUM] Project notice information can disappear before migration

**Location:** internal/boardengine/store.go:298

**Issue:** The project branch returns after deleting the retired keys and stores no replacement in `run_settings`.
Its notice survives only if reconciliation reads the original board before any save.
After an unrelated board save, even recovering folded task settings cannot recover this project's request.
That loss keeps F13 open separately from the task-value reader gap in F5.

## Validation

The named-package build passed for dobbyengine, hubgeom, configreg, landingshed and boardengine.
No target file was changed.
The four lens reports are saved beside this report.
The mutation candidates are logging statements and an allocation-capacity choice, not missing behavior assertions.
