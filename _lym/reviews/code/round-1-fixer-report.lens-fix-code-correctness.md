```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the code correctness of the fix (the `configuredBaselineReport` text change and the new `TestDriftBaseline` row for an existing `baseline.md`).

**Findings: no findings.**

1. `configuredBaselineReport` now renders "# Drift baseline", then "This report was generated, so no baseline agent ran and it expects no particular drift per test.", then "Scope: <scope>", plus "Skipped tests: …" when the skip list is non-empty. Only the prose sentence changed, so the signature and every caller are unchanged.
   - Input `ScopeRecord{Scope: full}` renders a report containing "no baseline agent ran" and "Scope: full". That is what the edited assertion in the configured-full row checks, and the gate passed.
   - The other rows (off, partial) still assert "Scope: off", "Scope: partial" and "Skipped tests: slow, flaky", which the unchanged lines still produce.

2. The new row "an existing baseline.md is kept under a configured scope, with no agent" seeds the full scope record with `report=true`, so `baseline.md` holds "expected drift". It then sets `configScope = full`. Run step by step:
   - `settleConfiguredScope` calls `UpdateScope`, finds the record and returns `errNoRecord`, which is swallowed.
   - The `os.Stat` of `baseline.md` succeeds, so the function returns nil without writing.
   - `drive` then finds the scope and the report, so no agent runs. The scope is full, `awaited_at` is empty, the lock is free and there is no capture record, so `captureAgain` runs and `StartCapture` is called once.
   - The commit is `scopeAndReport` and the agent count is 0, matching the row's expectations.
   - Removing the `os.Stat` guard would make `settleConfiguredScope` overwrite the file with the generated text. The `check` assertion `string(data) != "expected drift"` would then fail, so the row catches the defect. Seeded text and generated text differ, so the pin is real.

3. The `settleConfiguredScope` failure branches (an `os.Stat` error other than not-exist, `MkdirAll`, `WriteFile`) were not touched by the fix.

4. The stencil edit is prose only and carries no template marker. The added clause appends to the line after `{{.baseline_path}}`'s backticks, so it does not break the template.

Tool calls: none; my context already held every changed file, the review and the gate result.
