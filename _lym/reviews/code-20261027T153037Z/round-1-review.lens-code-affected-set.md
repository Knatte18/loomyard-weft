```yaml
lens: code-affected-set
outcome: reported
```

The affected set the diff leaves wrong is one stale doc line, `internal/reportengine/doc.go:131`; everything else I found is covered.

**Per change, the set I found:**
1. `darn-recipe.yaml` gains Drift-Await: loomrecipe tests, the loomcli checklist test, `internal/loomshed/interruptpolicy.go:58` (already covers `NameDriftAwait`), `internal/reportengine/timingspans.go` and its test table, the docs, and `driftengine/template.yaml:1` (already names darn) are covered. `reportengine/doc.go:131` is not, see finding 1.
2. `capture` key and `WaitCapture` removed: a repo-wide grep finds no `WaitCapture`, `AwaitRow` or `capture: wait`; `awaitCaptureEnd` is still used at `await.go:56`; `onSleep` and `sleeps` are still used by `await_test.go`.
3. `CountedCapture` loses `awaitRow`: callers `verify.go` and `verbs.go`, tests `records_test.go` and `verify_test.go`, and `driftcli/wiring.go` are covered.
4. `driftDeps` signature: its one caller `loomcli/wiring.go` is updated.
5. Drift CLI help text (`driftcli/cli.go:40`, `:260`, `:456-463`) needs no change.

**Findings:**

1. **LOW, class `consistency`:** `internal/reportengine/doc.go:131` still reads "A darn run's setup also holds Preflight and Drift-Baseline". Darn's Drift-Await now counts under setup too, per `internal/reportengine/timingspans.go:106` and `timing_test.go:312`. The loom sentence at `reportengine/doc.go:129` already names both Drift rows.

The decision record's assumption that the run report's phase map needs no darn-specific change was wrong. The `timingspans.go` edit is the required follow-through, not scope creep.

**Tool calls (2 of 6):**
1. A repo-wide grep of non-test files for "Drift-Baseline", to find docs, stencils and templates that list darn's rows.
2. Reads of `reportengine/doc.go:131` and the `driftcli/cli.go` help text.
