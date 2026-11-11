```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens on the codexengine probe change (F1 to F3).

**Findings**

1. **MEDIUM — the prose docs still say a probe error always degrades `auto`.**
   - Changed text: `launchSandboxMode` and `runSandboxProbe` in `sandbox.go`. The fix refuses the launch under both modes when the probe wraps `errSandboxProbeNotRun`.
   - Stale twins:
     - `doc.go` says "One that cannot start runs danger-full-access … under auto; under workspace-write it refuses the launch".
     - `docs/overview.md:700` says "`danger-full-access` with a warning when it cannot … an explicit `workspace-write` refuses the launch there".
     - `config.go` `sandboxAuto` says "danger-full-access when it cannot".
   - None of these mention the third outcome: a probe that did not run (missing or unexecutable binary, timeout, signal) refuses the launch under `auto` too. A reader of the docs would expect `auto` to degrade there.
   - The `sandbox.go` doc comments are updated; the `doc.go` and `overview.md` paragraphs that state the launch contract are not.
   - `template.yaml:6` (sandbox_mode comment) is truncated in my output, so I could not check it. It likely carries the same two-outcome description.

2. **LOW — the "way forward" in the not-run refusal contradicts the fix's own rationale.**
   - Changed text: the `errSandboxProbeNotRun` branch in `launchSandboxMode`.
   - It tells the operator to set `sandbox_mode=danger-full-access`. That drops the write bound, which is what F1 said a transient failure must not do silently.
   - The message does name "fix the Codex executable" first, so this is not wrong, only inconsistent in emphasis.

3. **LOW — a test comment is imprecise.**
   - Changed text: the `wantProbe` field comment in `TestOpenLaunch` (`open_test.go`).
   - It is reused as the "skip the home-unmade assertion" switch, so a future refusal row that sets `wantProbe` silently loses the check.
   - The field is documented only as "whether the launch probes", and the test doc explains the probe-after-home ordering. The coupling is documented in prose but not on the field.

Everything else is consistent. The `TestRunSandboxProbe` and `TestPrepare_SandboxMode` docs match their rows. The `sandboxProbeTimeout` comment now matches the not-run semantics. The `sandboxProbe` field doc in `codexengine.go:41` ("reports why Codex's sandbox cannot start") is still accurate for the refusal kind, though it does not cover the not-run kind.

**Tool calls**
- Call 1: grep for "cannot start" and "probe" across `docs`, `doc.go`, `config.go`, `codexengine.go` and `fixture_test.go`. My context lacked the sibling docs.
- Call 2: grep for `sandbox_mode` and "sandbox probe/cannot/can" in `overview.md`, `doc.go`, `config.go` and `codexengine.go`, to find the exact stale lines.
- Call 3: sed of `overview.md:700`, `doc.go:10-30` and `config.go:46-64`, plus a repo-wide grep for `sandbox_mode` in yaml and md. My context lacked those texts.
