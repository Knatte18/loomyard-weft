```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: what the regrouped `TestBattenIntegration_Rows` table and the `PATTERN-pair-remote.md` path change force outside the lines they touch.

**Per change, the set found:**

1. **Subtest paths moved from `TestBattenIntegration_Rows/<Step>` to `TestBattenIntegration_Rows/<Group>/<Step>`.**
   - `pattern/PATTERN-pair-remote.md:140`: updated to `.../CreateRow/RecordsPrimeRefusal`, which matches `lifecycle_integration_test.go:392,398`. Covered.
   - The test is named only at top level, never by subtest path, in:
     - `internal/battencli/wire_test.go:717`
     - `docs/benchmarks/test-redundancy.md:148-162`
     - `docs/benchmarks/test-suite-timing.md:299`
     - `lifecycle_integration_test.go:1196`

     None goes stale. Covered.
   - `docs/research/speed/batten-start-2026-10-18/report.md:10,17,62` and `docs/research/speed/test-time-measure-2026-11-27/report.md:81` name steps by their old flat name and describe the serial run. Both are measurement reports, which the prose rules exempt. Not stale.
   - A search of `.go`, `.md`, `.yaml`, `.toml` and `.json` files found no `-run` filter, gate card or tool config naming a Rows subtest path. Covered.
2. **The test's doc comment moves from steps serialised on one shared hub to parallel groups, a hub each.**
   - No doc, comment or spec outside the diff still says the Rows steps serialise on one hub. The "one shared hub" and "one hub copy" hits belong to other packages' own tests (`fabricengine`, `loomcli`). Covered.
3. **The new group subtests call `t.Parallel()`.**
   - Rule `PATTERN-test-speed`, "Tests run in parallel" (`pattern/PATTERN-test-speed.md:223-224`): a test that touches process-global state does not call `t.Parallel`, and a comment names that state.
   - Repository convention for `logcapture`: `internal/battencli/overview_test.go:694`, `internal/battencli/codetipsettle_test.go:22`, `internal/configreg/fill_test.go:142` and `internal/configengine/config_test.go:146,428`.
   - **Not covered:** see the finding below.
4. **Commit messages** (`git log 3fb0787e3..HEAD`, one commit `d9a84eb20`).
   - It has no `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailer, and none is needed: no assertion is loosened and no bug fix is claimed.
   - The commit message matches the diff: seven groups, every step kept, the dirtying step last in its group. Covered.
5. **Declared PATTERN changes:** the directive declares none. Nothing to check.

**Findings:**

- **severity:** MEDIUM
  **class:** design
  **location:** `internal/battencli/lifecycle_integration_test.go:412-413` (with the doc comment at 342-350)
  **summary:** The new `CommandRows` group subtest calls `t.Parallel()`, and its comment names no process-global state, yet one of its steps takes over the process-wide logger.
  **evidence:**
  - The step `stepCommandRows_IgnoreACommandSetInTheTasksTrackedLayer` calls `logcapture.Capture` (`lifecycle_integration_test.go:1395`).
  - `logcapture.Capture` redirects the process-wide logger and resets verbosity in cleanup (`internal/testkit/logcapture/logcapture.go:47-51`).
  - `PATTERN-test-speed` (`pattern/PATTERN-test-speed.md:223-224`) says such a test does not call `t.Parallel` and a comment names the state. The repository applies this to `logcapture` users: `overview_test.go:694`, `codetipsettle_test.go:22`, `configreg/fill_test.go:142`.
  - Under this diff, the six other groups run alongside that step's capture. Their logger output lands in its buffer, and its cleanup resets the logger under them.
  - The top-level test already called `t.Parallel` before this diff, so the violation predates it. The diff adds a new `t.Parallel` site over the step and rewrites the doc comment without naming the state.
  - Fix: move that step out of the parallel groups, or name the logger takeover in the doc comment and justify it.

Tool calls:
1. One search, run as a single command: a grep for "serialise", "one shared hub" and "one hub copy" wording, plus spans of `docs/benchmarks/test-redundancy.md` and `wire_test.go`, plus a grep for the step names outside the test file. My context had only the top-level `TestBattenIntegration_Rows` grep; it did not show wording that describes serial steps or step names cited as subtest paths.
