```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set. For each change, I listed what it forces outside the lines it touches. I found no defects.

Per change (set found, covered or not):
- **`RunView.EntryEnded` and `PairPresent`:** `battencli.DobbyRuns` (`dobbyreads.go:165-190`) is the only non-test constructor and sets both. `slotsTaken` (`starts.go:316`) is the only reader of `EntryEnded` in production code; `strandedEnded` also reads both fields. Covered.
- **`slotsTaken` rule change:** its callers are `runsInFlight` (`starts.go:303`) and `BuildView` (`view.go:119`). `BuildView` takes the same rule, so the view's `max_runs` reasons stay consistent. `view.go:93` says only that done runs take a slot while their driver lock is held, and it stays true. The `startReady` doc comment, `doc.go:204` and `doc.go:273` are updated. Covered.
- **New seam `Deps.TearDownEnded`:** `beamcli/dobbydeps.go:153` fills it. The other `Deps` constructors, `dobbyengine/passes_test.go` and `beamcli/dobbytrace_integration_test.go`, leave it nil, and a nil seam is documented as a no-op. Covered.
- **Binary check skipped while `d.teardown != nil`:** this is documented in `dobbyengine/doc.go:114` and noted in `loop.go`. Covered.
- **Docs:**
  - `dobbyengine/doc.go` and `beamcli/doc.go` describe the behavior.
  - The `dobbydeps.go` file header and the `ended_test.go` and `dobbydeps_test.go` headers are updated.
  - A grep of `docs/` and stencils found no other statement that halted runs count under `max_runs`; `docs/overview.md` mentions `max_runs` only for front tasks (line 819), which is unaffected.
- **PATTERN entries:**
  - `PATTERN-pair-remote`: teardown goes through `pairteardown.New` and `OriginSeams` with `HubSeamOptions`, the same route as the other hub-scope `ReconcileOrigin` callers. Covered.
  - `PATTERN-dobby-runs`: no violation.
  - `PATTERN-child-processes`: the goroutine starts no OS process of its own, and `ReconcileOrigin` is the existing fabric path. No violation.
  - `PATTERN-daemon-wakeups`: the loop stays busy only while a teardown is in flight, and retries back off to an hour. No violation.
- **Declared PATTERN changes:** the directive declares none, and the diff has no `PATTERN.md` or `pattern/` change. Nothing to raise.
- **Commit messages:** there are two commits, with no `Pattern-Departure:` trailer and none needed.

Findings: none.

Tool calls:
1. Grep for `NoSlot`, `slotsTaken`, `NoSlot:`, `Deps{`, `SpawnPass:` and docs mentioning `max_runs` or `halted`. My context lacked the full list of counters and `Deps` constructors.
2. Read `view.go` 95-135, plus a grep of `docs/overview.md` and `TearDownEnded` in `doc.go` and the integration test. I needed to know how `BuildView` uses `slotsTaken`.
3. Read `view.go` 84-95, plus a grep for slot-count statements in the docs. I needed to see whether the `BuildView` doc comment states which runs count.
