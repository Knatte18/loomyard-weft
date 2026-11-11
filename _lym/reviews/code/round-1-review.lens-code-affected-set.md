```yaml
lens: code-affected-set
outcome: reported
```

Task: affected-set review of the sandbox-mode change (lens code-affected-set).

**Per change, the set found:**
- **`codex.yaml` `sandbox_mode` gains `auto` (template default).**
  - Readers and asserters: `config.go` `LoadConfig` validation, `ConfigSpec` machine category, `TestLoadConfig_Template`. All updated.
  - `internal/configcli/configcli_test.go:1580` and `internal/configreg/configreg_test.go:147,193` assert the key name and the `danger-full-access` value only. Both stay valid. Covered.
  - Docs: `docs/overview.md`, `doc.go` and the `template.yaml` comment all describe the three modes. No other doc or rule document in `docs/`, `pattern/` or `README` names the key; `docs/research/**` hits are historical reports. Covered.
- **`Prepare` and `OpenLaunch` now probe, with a new `sandboxProbe` field.**
  - Constructors: `New` and `NewFromConfig` both set it. The only external constructor caller is `internal/shuttleengines/shuttleengines.go`. No `&Codex{}` literal exists elsewhere, so there is no nil-probe path. Covered.
  - Test fixtures: `newEngine` stubs the probe, so existing tests never run Codex. Covered.
  - Launch line: `command.go` still handles only `workspace-write` or `danger-full-access`, and `launchSandboxMode` never returns `auto`. Covered.
- **New refusal message in `launchSandboxMode`.**
  - Its way-forward text names `lym config codex --layer machine --set sandbox_mode=auto`. That matches the `--set` flag form `configcli_test.go:1580` uses. `PATTERN-refusal-way-forward` is satisfied, because the message names the way forward and `TestPrepare_SandboxMode` asserts it.
  - The refusal carries no `PATTERN-shuttle-provider-seam` problem: the probe sits under `internal/shuttleengine/codexengine`.
- **New process spawn in `runSandboxProbe`.**
  - `PATTERN-child-processes` is met: the spawn is logged through `internal/logger` and waited on by `CombinedOutput`. It is not a run process, so `runproc` does not apply.
  - `PATTERN-test-speed`: the tier-1 `TestRunSandboxProbe` carries `tiergate.Require`, and its non-parallel state is stated in a comment.
- **Declared PATTERN changes:** the directive lists none, and the diff touches no `PATTERN.md` or `pattern/` file. Nothing to land.
- **Commit messages and departures:** no `Pattern-Departure:` trailers. Nothing to check.
- **Stencils:** the diff changes none.

**Findings:** none from this lens.

Tool calls (2 of 6):
1. `grep` for `sandbox_mode`, `SandboxMode` and `danger-full-access` outside codexengine: to find external readers and asserters.
2. `grep` for `sandboxProbe`, `&Codex{` and sandbox uses in non-test codexengine files: to find other constructors and consumers of the mode.

I also ran a third combined search for external `codexengine.New` callers and `workspace-write` in docs and rules. This is the call-count mismatch: the first call above was a two-command batch, so the total was 3 Bash calls.
