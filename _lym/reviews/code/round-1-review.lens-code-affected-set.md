```yaml
lens: code-affected-set
outcome: reported
```

**Task:** code-affected-set lens. For each change the diff makes, the set it forces outside its own lines.

**Per-change list**

1. `SessionFold.settle` drops the unproven case, and `SessionCauseLivenessUnproven` is removed.
   - Covered: the callers `sessionread.go` (`readRunSessionState`) and `readiness.go` (`sessionReadiness`) compile unchanged.
   - Covered: the `sessionstate.go` comments, `doc.go:511`, `codexengine/doc.go:21` and `sessionstate_test.go`.
   - Not covered: `internal/battencli/wire_seams_test.go:415` still holds the string literal `"liveness-unproven"`. The cause no longer exists. See F1.
   - Covered: no other Go or doc file outside `docs/research` names the cause. That file is a measurement report and exempt.

2. The Claude and Codex probers still answer unproven, and a `Run.sessionGone` reader still treats unproven as "not proven gone".
   - `wait.go:1543-1559` is unchanged and its behaviour is unchanged. Covered.
   - `claudeengine/doc.go:214` describes the prober's answer, not the fold. Still true. Covered.

3. The hidden-anchor refusal in `Spec.validate`.
   - The only production caller is `run.go:559`. `Start` and `Run` reach it before any strand exists. Covered.
   - Covered: `spec.go:171` and `doc.go:79` carry the refusal.
   - Not covered: `attach.go:415` and `attach.go:559`, and `wait.go:1405`, keep the hidden branches. This is deliberate and recorded in the summary for old-binary run records. Attach never validates, so a hidden spec passed to Attach is not refused. That is consistent with the summary, so no finding.
   - Covered: `reedcli` and `reedengine` keep `hidden` for `lym reed add`. Out of scope.

4. The `lym shuttle run` flag, help text and refusal text.
   - Covered: the help text, the flag usage and `cli_test.go` (new `AnchorHidden` row).
   - Not covered: the refusal text for non-hidden bad values. See F2.
   - Covered: no stencil, spec, skill or `docs/overview.md` text advertises `--anchor hidden` for shuttle. I grepped `contracts`, `plugins` and `docs/overview.md`; the only anchor hits there were unrelated beam and reed ones.

5. Disagreement logging and readiness.
   - The `waitshadow.go:182` log and the `doc.go:528` text are consistent. Covered.
   - The `readiness.go` comment at line 87 ("An unknown state is neither: the caller falls back to the pane") is still true for the remaining unknown causes. Covered.

6. PATTERN and commit rules.
   - The diff makes no PATTERN change and the directive declares none. Nothing to land.
   - No `Pattern-Departure:` trailer is present and none is needed.
   - `PATTERN-refusal-way-forward` binds only the webster, shed and loom packages, not shuttle.

**Findings**

- **F1, LOW, consistency.** `internal/battencli/wire_seams_test.go:415-421`.
  - The test seeds `wantState, wantCause = "unknown", "liveness-unproven"` for the case where `proc.StartTime` is not readable.
  - After the diff that case reads `idle-stalled/no-output`, since the stamped Stop leaves no output, so the fallback expectation is false.
  - On Linux the branch is not taken, so the test passes. On a platform without `proc.StartTime` it fails.
  - The registry setup is also no longer needed to avoid `unknown`, and its comment is stale.
  - Fix: drop the `liveness-unproven` default and the registry setup, and expect `idle-stalled/no-output` unconditionally.

- **F2, LOW, consistency.** `internal/shuttlecli/run.go:122-124`.
  - The branch now covers every value except `own-window`, including `sideways` and `overview`.
  - Its message tells those users that "reed starts no command for a hidden strand".
  - That cause is wrong for them.
  - Fix: use the cause clause only when `anchor == string(render.AnchorHidden)`, and otherwise print the plain `want own-window` message.
  - The `AnchorUnknown` row in `cli_test.go:124` asserts only `own-window`, so it still passes.

**Tool calls**
1. grep over the shuttleengine docs and sources for `unproven` and hidden-anchor wording. My context had not covered `claudeengine/doc.go`, `sessionsignal.go` or `waitshadow.go`.
