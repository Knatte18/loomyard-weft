---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/shuttleengine/claudeengine/settings.go:337
    summary: Both Python hooks use a PATH lookup instead of the required selfexec binary resolver.
    fix: Resolve the lym executable through selfexec.LymBinary or an injected selfexec.ResolveLymBinary lookup at launch; pass its quoted path to both engines' Python hook renderers and manifest proofs, and supply a stand-in lookup in tests.
  - id: F2
    severity: BLOCKING
    class: consistency
    origin: lens:code-affected-set
    location: internal/shuttlecli/hookpython.go:40
    summary: The hidden verb emits plain-text errors contrary to PATTERN-cli-cobra.
    fix: Replace fmt.Fprintln with output.Err on cmd.ErrOrStderr(), preserve exit 1 and empty stdout, and update the existing failure row to assert the JSON error.
  - id: F3
    severity: BLOCKING
    class: consistency
    origin: lens:code-tests
    location: internal/shuttleengine/claudeengine/settings_test.go:397
    summary: The retained contained toggle test lacks the required keep marker.
    fix: Add a testtiming:keep directive immediately above TestBuildSettings_DenyToggleMatrix explaining that it pins the interactive AskUserQuestion recorder command and actual settings entries that TestPrepare_WritesTheManifest does not assert.
  - id: F4
    severity: BLOCKING
    class: consistency
    origin: lens:code-tests
    location: internal/shuttleengine/codexengine/guards_test.go:29
    summary: The retained contained guard-membership test lacks the required keep marker.
    fix: Add a testtiming:keep directive immediately above TestInstalledGuards_CoverEveryRow naming its independent row-membership and ordering assertions, which TestPrepare_InstallsGuardsFromItsInputs derives from installedGuards itself.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/shuttlecli/hookpython.go:26
    summary: A Python permission check can run the root stencil-seeding writer.
    fix: Add clihelp.SkipStencilSeedAnnotation with clihelp.AnnotationEnabled to hookPythonAllowedCmd's annotations so this read-only predicate skips root stencil seeding.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shuttlecli/hookpython_test.go:43
    summary: Config-load failure and missing-key default behavior have no hook regression rows.
    fix: Extend TestHookPythonAllowedVerb_FollowsTheHubsDenyPython with malformed or unreadable configuration and a missing deny_python key; assert exit 1, empty stdout, and a diagnostic only for the load failure, using separate output buffers where necessary.
  - id: F7
    severity: MEDIUM
    class: scope
    origin: lens:code-scope
    location: internal/shuttlecli/hookpython_test.go:59
    summary: The requested orch-launch regression test is missing.
    fix: Add an orch regression case that writes deny_python false at the hub default layer, follows normal orch config wiring into launch preparation, and asserts the Python notice is absent while its live hook remains installed.
---

### [BLOCKING] Resolve the hook executable through selfexec

**Location:** internal/shuttleengine/claudeengine/settings.go:337 and internal/shuttleengine/codexengine/guards.go:36

**Issue:** Both new commands embed bare `lym`.
`PATTERN-child-processes` explicitly requires hooks to resolve that binary through `selfexec`.
Existing PATH-based hooks do not exempt these new sites, and the commits carry no departure trailer.

### [BLOCKING] Preserve the CLI error contract

**Location:** internal/shuttlecli/hookpython.go:40

**Issue:** The failure branch prints a plain-text prefix.
`PATTERN-cli-cobra` requires errors through `internal/output`, with no exception for this hidden verb.
Writing JSON to stderr preserves the hook's empty-stdout contract.

### [BLOCKING] Record why the Claude candidate stays

**Location:** internal/shuttleengine/claudeengine/settings_test.go:397

**Issue:** The redundancy report lists this test as contained.
Its interactive marker command assertions are absent from its manifest-only covering test.
The rubric and `PATTERN-test-speed` require a keep directive when those assertions justify retention.

### [BLOCKING] Record why the Codex candidate stays

**Location:** internal/shuttleengine/codexengine/guards_test.go:29

**Issue:** This contained candidate independently checks row membership and order.
Its covering Prepare test calls `installedGuards` to build its own expected launch.
It therefore does not replace these independent assertions; retention requires the prescribed keep directive.

### [MEDIUM] Keep the permission predicate free of stencil writes

**Location:** internal/shuttlecli/hookpython.go:26

**Issue:** The command lacks the skip-seeding annotation.
The root pre-run calls `seedStencils` at `cmd/lym/main.go:184`.
When deployed content is due, `cmd/lym/stencilseed.go:144` pulls and commits Bolt writes before the predicate answers.
The module-only test bypasses that root behavior.

### [MEDIUM] Cover the config failure branch

**Location:** internal/shuttlecli/hookpython_test.go:43

**Issue:** Valid boolean values and a non-repository cwd are covered, but config-load errors and default filling are not.
The engine shell tests stub the lym exit status and cannot detect a regression in the CLI's load-error handling.

### [MEDIUM] Exercise the requested orch launch

**Location:** internal/shuttlecli/hookpython_test.go:59

**Issue:** The new hub test calls the hidden predicate alone.
Engine tests construct launch config values directly.
Neither exercises the requested default-layer-to-orch-notice path.
The amendment changes the hook expectation, but leaves the launch-notice requirement intact.

The existing orch wiring already uses `fabricengine.ConfigDirs(location)` and passes the loaded shuttle config onward.
No production config-routing change is needed on the evidence reviewed.
The live checks and launch-time notice filtering otherwise implement the amendment in both engines.
The installed `lym` rejected the new verb as unknown, so the runtime probe did not execute this branch's implementation.
No build or test failure is asserted by this review.
