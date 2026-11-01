```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F1–F4).

**Findings**

- **MEDIUM**: `internal/driftengine/template.yaml`, the added text "or one key alone with lym config drift --set capture.timeout_min=30" on both the `capture: {}` and `compare: {}` default lines.
  - **Summary:** On the default undeclared step, the advertised single-key set is refused by the load check, so the comment promises a behavior the code does not hold in that state.
  - **Evidence:** Setting `capture.timeout_min=30` on `capture: {}` writes `capture: {timeout_min: 30}`. `decodeStep` (`internal/driftengine/config.go`) treats a step with any key set as declared and refuses it with `capture.command: missing`. The doc in `internal/configcli/doc.go` says every `--set` write is checked through `CheckLoad` and rolled back on a load error.
  - **Fix needed:** The comment sits on the `{}` default line, so a reader tries exactly this. It is only valid once a step is declared, or when `command` is set in the same call.
  - **Not covered by the test:** `TestConfigSpec_SetNestedKeys` seeds a declared capture, and it sets `compare.command` and `compare.timeout_min` together, so neither case runs the undeclared single-key path.

- F1 (shape for capture and compare): no other breakage found. `ShapeOf(stepFile{})` gives `command`, `dir` and `env` as strings or a map of strings, and gives `timeout_min` (a `*int`) as a number. That matches `decodeStep`.
- F2, F3, F4 (new rows and the new test): no finding. I built the cases below and none broke.
  - The `Covers []int` and `Plain`, `Retries`, `secret` additions change the sorted-key list in the `envv` refusal row, and the fixer updated that string.
  - `var _ = shapedTier{secret: ""}` is harmless.
  - The all-or-nothing test relies on `shapedKeyError` running before any pair applies, as the `SetValues` code shows.

**Tool calls:** none. My context already held every file the lens needed.
