```yaml
lens: repo-invariants
outcome: reported
```

Task: repo-invariants lens over cards 1–13 (test seams and tier tags, docs a card makes false, card order).

**Per-card list of invariants checked**

- **Card 1**
  - Seams: no reflective test covers `dobbyengine` `Config` or beamsetting. Held.
  - Docs: card 1 edits the beamsetting package comment, dobbyengine's `doc.go`, `tasksetting.go`'s file comment and `PATTERN-config-layers.md`. Held.
  - Order: the classification test (configreg_test.go:212-223) lists machine keys only. `beam.window_placement` is `CategoryNone` and card 1 classifies it. Held.
- **Card 2**
  - Order: `wantMachineKeys` at configreg_test.go:217 is edited. Held.
  - Docs: `window.go`'s package comment ("Both keys are machine keys") stays loosely true until card 12 rewrites it. Held.
- **Card 3**
  - Tier: `TestAfterPlacement` (tier1) goes in desktophook `window_test.go`, a tier1 file under `!windows`. Held.
  - Helper: `newDirsWithMachineConfig` exists (machine_test.go:15). Held.
  - Docs: the package comment (window.go:4 "It waits desktop.yaml's window_wait_ms") is false for `AfterPlacement` from card 3 until card 12 edits `window.go#`. Card 12 covers it. Held.
- **Card 4**
  - Tier: the tier1 test goes in `termwindow_integration_test.go`, a tier1 file. Held.
  - Seams: the new `Launcher` field has no reflective test. Held.
  - Docs: the termwindow package doc ("lym neither waits for it nor learns whether it stays open") stays true. Held.
- **Card 5**
  - Tier: `testmain_test.go` is untagged and support-only. Held.
  - Tier: the tier0 tests name no file. See card 7.
- **Card 6**
  - Order: uses `Launcher.Open` from card 4 and the package from card 5. Held.
- **Card 7**
  - Tier: the tier3 contract test shares a package with tier0 tests and has no file named. **Broken risk (F1).**
  - Docs: `docs/benchmarks/running-tests.md` describes tier3 as LLM-only. **Broken (F3).**
  - Daemon wakeups: the polls run through the sleeper field, not a ticker, so the NewTicker scan holds. Held.
- **Card 8**
  - Lock path: `places.lock` under `BoltDir/.lym/window-places` passes `refuseTrackedTree`, which resets `inBolt` at `.lym` (lock/tracked.go:77-81). Held.
  - Paths: built from lymdirs, and the lymdirs `doc.go` is generic. Held.
- **Card 9:** pure functions, no file-level invariant at stake. Held.
- **Card 10**
  - Daemon wakeups: `parentwait.go` joins `unboundedWaitFiles` and the background list. Held.
  - Seams: the new `Flow` and `Runner` structs have no reflective test. Held.
- **Card 11**
  - Docs: edits reedcli's `cli.go` file comment, its pre-run comment and `doc.go`. `RolePlacement` keeps runproc's `doc.go` ("one of the `Role` constants") and `PATTERN-child-processes` true. Held.
  - Tier: `TestPlaceWindowChild` (tier1) has no file named. **Risk (F2).**
  - Process-global PATH in a top-level non-parallel test. Held.
- **Card 12**
  - Seams: the new `reedCLI` field `handPlacement` is reflected over only by `reflect.DeepEqual` (down_test.go:161, oomnotice_test.go:78), with no `//lym:seam`. Held.
  - Docs: reedcli `window.go`'s file comment becomes false. **Broken (F4).**
  - Tier: the tier1 test has no file named, and `window_test.go` is tier0. **Risk (F2).**
- **Card 13**
  - Seams: the new `deps` field is reflected over only by `reflect.DeepEqual` (start_test.go:696). Held.
  - Docs: edits machinecli `doc.go`, the doc comments on `deps` and `startSession`, and docs/overview.md. Held.

**Findings**

- **F1 — MEDIUM, consistency, `_lym/plan/07-kwinplace-script-runner.md`**
  - Summary: card 7 adds the tier3 `TestContract_TheRealKWinAnswersAsRecorded` to `internal/kwinplace` beside its tier0 runner tests. Neither card 7 nor cards 5, 8, 9 and 10 name a test file, so nothing keeps the tier3 function out of a tier0 file.
  - Rule broken: PATTERN-test-speed's "The functions of one test file name one tier", enforced by `cmd/lym/tiertags_test.go`.
  - Fix: card 7 names its own tier3 file for the contract test, e.g. `internal/kwinplace/contract_test.go`, and states the file of its tier0 tests.
  - Evidence: pattern/PATTERN-test-speed.md, "Four tiers" bullets ("The functions of one test file name one tier … Enforced by cmd/lym/tiertags_test.go"); card 7 lines 513 and 535.
- **F2 — LOW, consistency, `_lym/plan/12-reed-window-placement.md` and `_lym/plan/11-place-window-child.md`**
  - Summary: card 12's tier1 `TestHandTerminalWindow_DirectoryInNoWorktreeHandsNothing` and card 11's tier1 `TestPlaceWindowChild` name no file. Card 12 puts its tier0 rows in reedcli `window_test.go`, which is tier0, so the natural home for the tier1 test breaks the one-tier-per-file rule.
  - Fix: name a tier1 file, such as the existing `internal/reedcli/placewindow_integration_test.go` (tier1, `!windows`) or a new tier1 file.
  - Evidence: internal/reedcli/window_test.go (`tiergate.Require(t, 0)` only); internal/reedcli/placewindow_integration_test.go:1 and :23 (`tiergate.Require(t, 1)`).
- **F3 — LOW, consistency, `_lym/plan/07-kwinplace-script-runner.md`**
  - Summary: card 7 puts a real-desktop test in tier3 and edits only PATTERN-test-speed's table. `docs/benchmarks/running-tests.md` still defines tier3 as "tests that spawn a real `claude` session" and "real LLM sessions", and no card targets that file.
  - Fix: add `//docs/benchmarks/running-tests.md` to card 7, with the desktop session added to both lines.
  - Evidence: docs/benchmarks/running-tests.md:34 and :55.
- **F4 — LOW, consistency, `_lym/plan/12-reed-window-placement.md`**
  - Summary: card 12 changes `windowCmd` so that under placement the child, not the verb, runs the window command. Its targets list `openTerminalWindow`, `windowCmd` and `reedCLI` but not `internal/reedcli/window.go#`, so the file comment ("It then runs the machine's window command through internal/desktophook") becomes false.
  - Fix: add `internal/reedcli/window.go#` under `**Edit:**` and say the comment names the hand-off to the placement child.
  - Evidence: internal/reedcli/window.go:1-5; card 12 lines 876-883.

**Tool calls**

1. Grepped for seam markers, the tier calls of the touched packages' test files, `newDirsWithMachineConfig`, `refuseTrackedTree` and pause-lock paths: the manifest held none of the tier tags, the lock guard or the helper.
2. Grepped for reflection in the reedcli and machinecli tests, read lymdirs `doc.go` and the configreg classification rows: the manifest named those test files but did not read them.
3. Listed the reflect call kinds and found the tier3 and `window_wait_ms` doc sites: needed to tell DeepEqual from a field walk and to find docs outside the plan.
4. Read the tier3 lines in other pattern files and docs: to confirm which ones card 7 makes false.
5. Checked gateslot's tier `about` and the daemon-wakeups mentions: to see whether a tier description in repo code needs a change (the hub's gate.yaml is hub config, not a card target).
