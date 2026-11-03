```yaml
lens: code-correctness
outcome: reported
```

Task: judge whether the code the diff changes is correct, by running concrete cases through each function and branch (lens: code-correctness).

**Per function or branch: case run, verdict**

- `drive`, new `d.Config()` call. Case: drift off. `Call` returns at `passWhenSwitchedOff` (`baseline.go:62`) before `drive`, so the new call is not reached with drift off. Nothing is spawned, started or committed, and `Call`'s doc comment still holds. Correct.
- `drive`, `d.Config()` error. Case: the config fails to load a second time. It returns Stuck with "then lym loom resume", the same shape as `passWhenSwitchedOff` and `captureAgain`. Correct. It loads the config three times per pass (`passWhenSwitchedOff`, `drive`, `captureAgain`), which is harmless.
- `drive`, `cfg.Scope == ""`. The path is unchanged from before and reaches the agent branch. Correct.
- `settleConfiguredScope`, no scope record, `cfg.Scope = full`, `Skip = nil`. `UpdateScope` writes `{Scope: full, Skip: null}`. This is the same shape as `RecordScope` with an empty skip. `baseline.md` is then written. Correct.
- `settleConfiguredScope`, no record, `partial` with `Skip = [a, b]`. The record holds the skip list and the report prints it. Correct.
- `settleConfiguredScope`, record exists (an earlier pass, or a parent replacement at a blocked Drift-Baseline or Drift-Await). The mutate returns `errNoRecord`, and `state.UpdateJSON` writes nothing when mutate errors (the `UpdateScope` doc says so, and `CloseAgentSession` at `records.go:252-262` relies on the same pattern). `errors.Is` swallows only that sentinel; any other error from `UpdateScope` is returned. Correct. The recorded scope wins over `cfg.Scope`.
- `settleConfiguredScope`, record exists but `baseline.md` is missing. This covers a crash between the two writes, or a parent replacement before the report existed. The report is written from the recorded scope, not from `cfg.Scope`, and the state stays consistent. Correct. Residual issue below (finding 1).
- `settleConfiguredScope`, `os.Stat` fails with something other than not-exist. It returns a wrapped error, which is an error outcome, not a Stuck. Correct.
- `settleConfiguredScope`, `ReadScope` after `UpdateScope`. The record is always found at that point, and the `found` flag is ignored. Correct, because the update just ensured a record exists.
- `settleConfiguredScope`, `MkdirAll` and `WriteFile`. These are plain, non-atomic writes. A crash mid-write could leave a truncated `baseline.md`, which a later pass would treat as present. `RecordScope` and the agent path have the same exposure. Not a finding.
- `drive` after the settle: `ReadScope` finds the record and `Stat` succeeds, so the agent branch is skipped. With scope `off` it returns Done. `partial` and `full` reach the capture logic unchanged. Correct.
- Failure path: if `settleConfiguredScope` errors, `drive` returns an error outcome and `commitOnExit` commits the scope and report paths. The state is whatever was written, which is consistent with the above. Correct.
- `configuredBaselineReport`, `Skip` empty or nil. The "Skipped tests:" line is omitted. Correct.
- `checkScope`, `scope = ""` with `skip = [x]`. The third case matches (`"" != partial` and `len > 0`) and refuses. A skip with no scope is refused, which is consistent with the stated rule.
- `checkScope`, `scope = partial` with `skip = []` or nil. It is refused by `len == 0`. A non-nil empty slice is treated the same as nil. Correct.
- `checkScope`, duplicate skip entries. They are accepted, the same as `RecordScope` accepts them. Not a defect.
- `LoadConfig`, `Config.Skip`. It is set only under `partial`, and nil otherwise. Under `off` or `full` a non-empty skip is refused earlier. Correct.
- `LoadConfig`, run_settings path. `drift` is registered with `CheckLoad: checkLoad(driftengine.LoadConfig)` (`configreg.go:133`), so an invalid scope in `run_settings` is surfaced through the module's own loader. The two keys are `CategoryNone`, which the local layer accepts, and the test row in `configreg_test.go` pins them. Correct.
- Merging a local `skip` list over the template's `skip: []`. The local list replaces it. The new `TestLoadConfig` row covers a local `partial` scope with its skip list. Correct.

**Findings**

1. LOW, class `consistency`. `baseline.go:171` (`configuredBaselineReport`). The report always says "The drift config set the regression scope (`drift.scope`), so no baseline agent ran". `settleConfiguredScope` writes it from the recorded scope, and that scope can come from a parent's `lym drift scope` replacement or an earlier record that differs from `cfg.Scope`. In that case the text names the wrong source, and Drift-Verify's agent reads this file as its expectation. The `Scope:` line is still correct. Evidence: `baseline.go:140-176`, the `settleConfiguredScope` doc ("A scope already recorded, by an earlier pass or the run's parent, is kept").

No other defects found.

**Tool calls**
1. `grep -rn CheckValues internal --include=*.go`: to see whether run_settings values are validated at submission; my context lacked `configreg.go`'s registry.
2. `grep -n '"drift"' internal/configreg/configreg.go` and `grep -n CheckLoad ...`: to confirm that drift is registered with `CheckLoad: checkLoad(driftengine.LoadConfig)`; my context lacked that registry row.
