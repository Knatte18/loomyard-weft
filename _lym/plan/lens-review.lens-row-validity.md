```yaml
lens: row-validity
outcome: reported
```

Row-validity lens: whether each planned test row reaches the branch it claims, fails if the code is wrong, and is needed.

**Findings**

1. **BLOCKING — Card 25, `VerbStatus` loses `AgentSession`.** The test helper `statusAt` still sets that field and is not on card 25, so the package's tests stop compiling at card 25.
   - Evidence: `internal/driftengine/verbs_test.go:20-24`, glyph `internal/driftengine#statusAt`, sets `AgentSession: agentSession`.
   - Card 25 edits `VerbStatus` and `TestDeferWaive` but lists neither `internal/driftengine#statusAt` under Edit nor the rows that pass its fourth argument. Card 19 lists `statusAt` under Uses only.
   - Fix: add `internal/driftengine#statusAt` to card 25's Edit and drop its `agentSession` parameter.

2. **BLOCKING — Card 24 drops `VerifyTable`, `CompareAccepted` and `BaselineTable` from `Deps`.** `internal/shedrecipe#driftTestEnv` still sets them, so shedrecipe's tests stop compiling.
   - Evidence: `internal/shedrecipe/entries_drift_test.go:36` sets `VerifyTable` and `:40` sets `CompareAccepted`. The `BaselineTable` fill sits above line 36.
   - Card 24 edits loomcli, envkit and battencli, but not `internal/shedrecipe#driftTestEnv`. Card 16 edits `driftTestEnv` before the fields go, so it cannot drop them.

3. **MEDIUM — Cards 7, 8, 10, 13, 16, 19 and 20 each test against "a fake in-memory commit store" behind `CommitWithTrailers`, `TrailerCommit` and `FileAt`.** No card declares that fake as a `Create` handle or names it under Uses, so each card would either rebuild it or depend on an undeclared helper.
   - Evidence: card 7 creates `TestRecords` "with a fake in-memory commit store" but declares no helper handle. Cards 8, 13 and 16 repeat the phrase, and cards 16 and 19 lean on it implicitly.
   - Card 19's `TestDeferWaive` rows now call `Defer(d Deps, …)`, but the existing rows build only `Paths` through `seedScope` (`verbs_test.go:512-513`). Nothing listed gives them a `Deps` with those seams.
   - Fix: declare one shared fake and its `Deps` builder on card 7, and list it under Uses on the later cards.

4. **MEDIUM — Card 9's `TestCaptureCurrent` row "with InheritEnv naming a held slot … finishes without a second slot" cannot run under the test's stated `t.Parallel`.**
   - The card reads the slot from the process environment (`gateslot.InheritEnv = "LYM_GATE_SLOT"`, `internal/gateslot/pool.go:25`).
   - Setting it in a test needs `t.Setenv`, which panics in a parallel test.
   - Fix: carry the inherited slot on `CaptureRequest` or `stepRun` as a told value, or make that row non-parallel and name the process-global state, as `PATTERN-test-speed` asks.

5. **MEDIUM — Card 22 rewrites `TestDrift_RunsInTheAnchoredDirectory` to run `check` where it ran `compare` and `capture --side`. As specified, `check` never runs a command in that test.**
   - Evidence: the test writes a tapestry seed and a bare `Status{CurrentProducer: row}` (`internal/driftcli/anchor_integration_test.go:71,80`), and loops over `{"compare"}, {"capture","--side"}` (`:95`).
   - Card 10's `Check` stops before any capture when no scope record exists (`Skip`), when no capture counts (`BaselineFault`), or when the row is not past Drift-Expect (`NotYet`).
   - The card adds no scope record, `awaited_at`, recorded `Diff`, or a status row past Drift-Expect. The assertion that the command ran in the anchored directory would then fail.

6. **MEDIUM — Card 12's `TestCheck_PrintsTheFindings` needs the same state, which the card does not list.** It needs a counted capture, a scope record, and a seed whose recipe puts the status row past Drift-Expect, because `checkDeps` reads `Rows` from `shedrun.ReadSeed`. Without them the "changed value printed" row reads `NotYet` or `BaselineFault`.
   - Its Uses names only `driftYAML` and `lymRun`. The seed-writing step sits inline in the anchor test (`anchor_integration_test.go:71`), and no helper for it is named.
   - Fix: state the fixture's scope record, `capture.json`, `awaited_at`, seed and status row on the card.

7. **LOW — Card 18's `TestDriftBaseline` row "a config `partial` records `full` and warns" asserts a `logger.Warn`.** The only way to capture one is `logger.SetOutput` (`internal/logger/logger.go`), which is process-global, while the drift tests run `t.Parallel`.
   - Either the row cannot assert the warn, or it must be non-parallel and name the global.
   - Fix: assert the recorded scope only, or have the row write the warn through a seam.

8. **LOW — Card 8's `TestSealAndBaseline` row "editing a worktree file leaves the recorded digest unchanged" repeats the "edited baseline file faults" row in the same table.**
   - `CheckBaseline` recomputes the directory's digest and compares it with the trailer. Editing a worktree file changes neither input, so the row asserts nothing a broken implementation would fail.
   - Fix: drop the row, or state the input that would change the digest if it were read from the files.

9. **LOW — Card 26 adds the `{skip_file}` refusal in `renderCommand` and tests it only through `TestRunCapture` and `TestRunCompare`.** `CaptureCurrent`, which card 26 also edits, is the path the checks run, and no row reaches the refusal through it (D15 criterion "a capture command holding `{skip_file}` is refused").
   - Fix: add a `TestCaptureCurrent` row.

**Tool calls**
- Ran a grep over `orchcli`, `hubgeom`, `statusAt` and `TestDeferWaive`, because my context lacked the test helper's fields and the `Defer` rows.
- Read `entries_drift_test.go:36-100` and `gateLoopSeats`, because I lacked `driftTestEnv`'s seam fills and the shape of the entries table.
- Grepped `runsettings.go`'s order, `InheritEnv` and the driftcli fixtures' seed and status setup, because I lacked the migrate and validate order and the anchor test's setup.
- Grepped the logger for capture hooks and read `baselineFixture`, because I lacked the logger's test API and the fixture's fields.
