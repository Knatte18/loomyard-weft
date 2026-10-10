---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/websterengine/cardgate_test.go:173
    summary: TestRenderCardGates_OneBulletPerCardWithAStepSubBulletEach and TestCardGateCommand were rewritten in the range, touch only t.TempDir, and neither call t.Parallel nor carry a comment naming global state
  - id: F2
    severity: LOW
    class: consistency
    location: internal/gatecli/run.go:57
    summary: an absent gate.yaml reaches the CLI envelope with two way-forward commands and no gatecli test reaches that trigger of the spec row
---

### [BLOCKING] Rewritten websterengine card-gate tests neither run in parallel nor name global state

**Location:** `internal/websterengine/cardgate_test.go:27` (`TestCardGateCommand`) and `internal/websterengine/cardgate_test.go:173` (`TestRenderCardGates_OneBulletPerCardWithAStepSubBulletEach`).

**Issue:** `PATTERN-test-speed` requires a test to call `t.Parallel` unless it touches process-global state (env, cwd or a shared fixture), and requires a comment at the test or atop its file to name that state.
The diff rewrites both tests' bodies, and the second one is renamed, so it reads as added.
Both call only `cardGateCommand` and `renderCardGates` with `t.TempDir()` roots and the `writeGateFile` helper; there is no env, cwd, logger or shared fixture.
Neither test, nor `cardgate_test.go`'s header, carries a comment naming any state.
The other tests the diff adds in `internal/websterengine` (`TestRenderBatchGate`, `TestRerunCardVerifies_Slotted`) and in `internal/landingshed` (`TestPublishVerify_FailureRecord`, which names `NewGitHubClient` as the global state it swaps) comply, so these two are the only ones in the range that break the rule.
The constraint makes any violation BLOCKING; the omission pre-dates the range, but the range rewrites the tests.

**Fix:** add `t.Parallel()` to both top-level tests and to their subtests, which share nothing but their own `t.TempDir()`.

### [LOW] Absent gate.yaml: two way-forward commands in one envelope, and no CLI test reaches that trigger

**Location:** `internal/gatecli/run.go:57`, `internal/gateslot/config.go:29`, `internal/gatecli/test_integration_test.go:396`.

**Issue:** the `## gate` row "gate.yaml unreadable or invalid" in `contracts/specs/refusal-spec.md` lists three triggers (absent, does not parse, value below 1) and one way forward, `fix the file with "lyx config gate" from the prime`.
For an absent file, `gateslot.LoadConfig` returns `gate config absent; run "lyx fabric reconcile"`, which `run.go` wraps into `cannot read the hub's gate limits: ...; way forward: fix the file with "lyx config gate" from the prime, ...`.
The envelope then carries two commands for one trigger, and `lyx fabric reconcile` is absent from the spec row.
The scenario's refusal table reaches only the value-below-1 trigger, so the absent trigger's envelope is pinned by `gateslot`'s `TestLoadConfig` for the inner text alone, never as the CLI envelope the spec row describes.
The absent file is detected by `os.Stat` and `errors.Is(err, os.ErrNotExist)`, not by error text, so the detection itself is sound.

**Fix:** pick one way forward for the absent trigger and make the spec row, the `LoadConfig` message and the CLI envelope agree on it, then add the absent file as a row of the existing refusal table in `TestGateTest_Scenario` (its setup removes `gate.yaml`, its restore rewrites it) rather than a new test.

## Focus verdicts

Each directive question is settled here; none is a finding unless listed above.

- **gatecli refusal rows.**
  The rows with a test that reaches them: no package and `-C` missing in `cli_test.go` (`TestGateTest_RefusesBeforeResolvingTheHub`), slot busy, the value-below-1 `gate.yaml`, slot acquire failure and a `go` that cannot start in `TestGateTest_Scenario`.
  The absent `gate.yaml` trigger is F2.
  No test calls `goTestArgs` or another unexported helper directly; the scenario asserts the argument list through the stand-in's record, so there is no per-helper duplicate.
- **`t.Parallel` in websterengine and landingshed.**
  `TestRenderBatchGate` and `TestRerunCardVerifies_Slotted` call `t.Parallel`; `TestPublishVerify_FailureRecord` is serial with a comment naming the swapped `NewGitHubClient`, and its file header repeats the reason.
  The two rewritten tests in `cardgate_test.go` are F1.
- **`checkedPublishFailure` and `failedTestPackage`.**
  A record's log path is kept only when its cleaned form equals `paths.PublishFailureLog`, so no free file name reaches the prompt.
  A test package must start with a letter or digit and holds only import-path characters, so it cannot read as a flag, and each `-run` segment is anchored and quoted with `regexp.QuoteMeta`.
  Not a defect.
- **`gateslot.LoadConfig` absent file.**
  Recognised by stat and `errors.Is`, never by matching error text.
  Not a defect; the message shape is F2.
- **`configcli` menu write guard.**
  `TestConfigOverRealHub`'s menu row drives `Command()` through `menu` from a task pair with a no-op editor and asserts exit 1, the `prime worktree` way forward and an unchanged hub file; with `hubWriteGuard` unset the menu reaches `editHubWide` and the row fails on the exit code and the missing refusal.
  Not a defect.

## Verification run

Tier 1 over `gatecli`, `gateslot`, `loomshed`, `configcli`, `websterengine`, `landingshed`, `verifytree`, `planparser`, `commentlint` and `cmd/lyx` passed; tier 2 (`-tags integration`) over `gatecli`, `configcli`, `gateslot` and `loomshed` passed.
