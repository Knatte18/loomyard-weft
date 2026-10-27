```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. Per behaviour, the test that pins it, or none.

**Per behaviour:**
- **Drift-Baseline detaches its capture and returns Done once the child's record exists:** `TestDriftBaseline` rows with `wantStarts: 1` (`baseline_test.go:~192`). Pinned.
- **A held capture lock returns Done without waiting:** the "a held lock is left running" row, which asserts no sleeps (`baseline_test.go:~228-237`). Pinned.
- **Breakaway refusal runs the capture in-process, and a failed in-process capture is Stuck naming the ways forward:** the rows at `baseline_test.go:294-311`. Pinned. The removed waited-failure row was redundant with these.
- **The `capture` key is refused naming the way forward:** `TestDriftEntries` row `BaselineCaptureKey`, expecting `entry_gate` (`entries_drift_test.go:~64`). Pinned. The `driftKeyWayForward` text is the only way forward it asserts.
- **`CountedCapture` counts only a passed record with `awaited_at`:** `TestCountedCapture`, five rows (`records_test.go`). Pinned.
- **`lym drift waive` and Drift-Verify judge a counted capture by `awaited_at` on every recipe:** `TestDriftVerify` "no counted capture is a decision halt" rows (`verify_test.go:~265`). Pinned.
- **Darn's recipe is Preflight, Drift-Baseline, Drift-Await, Darn-Write, Drift-Verify, with the right edges:** `wantDarnProducerTable` and `TestNewDarn_Routing` (`darnrecipe_test.go`). Pinned.
- **Darn's Drift-Await carries `entry_gate: drift.enabled`, so every Drift row is skipped under the template:** `TestDriftRows_EntryGate`, darn rows (`drift_test.go:29`). Pinned.
- **Darn row numbers:** `TestEntryNumbers` (`recipe_test.go:362`). Pinned.
- **The checklist renders Drift-Await in darn:** `TestRenderChecklist` (`checklist_test.go:~490`). Pinned.
- **The report phase of the darn Drift rows:** `timing_test.go` `darnRowPhases` (Baseline and Await in setup, Verify in review) and `TestComputeTiming`. Pinned.
- **`verbStatus` no longer reads the seed:** none directly. The `driftcli` wiring has no test in this diff. It is a removal of an unneeded read, so I don't raise it.
- **Wiring drops `awaitRow` (`driftDeps`, `wire`):** none, and none is needed; compile-time only.
- **The recipe header comment and the docs:** none, and none is needed; static prose.

**Findings:**

1. **LOW, class `consistency`: `baselineFixture` likely carries dead Sleep scaffolding.**
   - Location: `internal/driftengine/baseline_test.go:42-49` and `:108-110`.
   - Summary: Drift-Baseline no longer reads `Sleep`, and the only rows that set `onSleep` were removed. The "held lock" row still asserts `len(f.sleeps) != 0` at `:236`, so that guard has a live reader. `onSleep` has no setter left in this file and is dead. I did not check other test files for a setter. `Deps.Sleep` itself stays, since Drift-Await uses it.
   - Evidence: `grep` shows `onSleep` at `:43`, `:109` and `:110` only.

2. **NIT, class `consistency`: the generic unknown-key row is gone.**
   - Location: `internal/shedrecipe/entries_drift_test.go:~64`.
   - Summary: `BaselineUnknownKey` (`"enabled": true`) was replaced by `BaselineCaptureKey`. Both go through the same `configRejectUnknown` branch, so nothing is lost. I would not require a fix.

No redundant new tests: every added row follows a changed recipe shape or replaces a removed one. Nothing was added for a non-gap review finding.

**Tool calls:**
1. `git diff` over the test files and the three source files: the manifest held no test diffs.
2. `grep` in `baseline_test.go` for `ErrBreakawayRefused`, `runRecord`, `onSleep` and `sleeps`: to confirm the breakaway and failed-capture rows survive and to find the dead scaffolding.
