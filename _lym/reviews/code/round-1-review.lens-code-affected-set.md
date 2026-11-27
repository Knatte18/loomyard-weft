```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change in the diff, I list what reads, calls or describes it and whether the diff leaves that correct.

**Per change: the affected set, covered or not**
- **`sessionLiveness` / `sessionRegistrar` move to `reedengine.SessionLiveness` / `SessionRegistrar`.** Callers `loomcli/sharedbootstrap.go:132-142` are updated. The landingshed session world uses the new names, and `TestPRSessionSeams` moves to `reedengine/sessionreads_test.go`. A repository search finds no stale reference in Go, docs or patterns. Covered.
- **The registrar's error prefix changes from `loomcli:` to `reedengine:`.** Nothing asserts the old text (search found nothing). Covered.
- **`paneProcessAge` is exported as `PaneProcessAge`.** The Linux and Windows definitions, `lock.go`, `proctree_test.go` and `ownwindow_integration_test.go` are all updated, and no stale reference remains. Covered.
- **`newGUID` and the memory cap now go through engine fields (`strand.go`, `panememory.go`).** Their only call sites are updated. Covered.
- **New `NewOver` / `Outside`, and the `toldTmux` spawn branch.** `reedengine/doc.go` documents both. The `execHook` and `floorLine` fields are reused, and existing reedengine tests still set them directly. Covered.
- **New kit `internal/testkit/tmuxrec`.**
  - `pattern/PATTERN-test-stand-ins.md` names it (one paragraph added).
  - `docs/overview.md`'s kit list (around :2096-2105) does not name it. Finding F2.
  - The kit's non-test imports (`proc`, `reedengine`, `configengine`, `lymdirs`, `shuttleengine`, `shuttlefake`) hit no ban in `enforcement_test.go`. Covered.
  - `PATTERN-test-isolation`'s Enforcement section, which lists the kits allowed to assert, does not cover what `PlaySession` does. Finding F3.
- **The tmuxrec test file with two tiers (`recordings_test.go`).** `PATTERN-test-speed` requires one tier per file and no tmuxkit spawn or `exec.Command` in a tier0 file. The guards that would catch it are `//lym:only parent-align finalize`, so no card gate ran them. Finding F1.
- **landingshed session world (`prreview_session_test.go`, `prreview_wait_test.go`, `prreview_test.go`).** Every use of the world and its `seedLive` signature is updated. The research inventory (`docs/research/no-fake-other-modules.md`) lists no landingshed session fake that would now be stale, and its `envkit` row is unchanged. Covered.
- **Declared PATTERN changes.** None are declared in the directive, so there is nothing to check. The `PATTERN-test-stand-ins` paragraph is undeclared. That is scope, and the code-scope lens owns it.
- **Commit messages and departures.** The range carries no `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailer, and none applies apart from F1.

**Findings**

- **F1 — BLOCKING, design.** `internal/testkit/tmuxrec/recordings_test.go`
  - **Summary:** one test file holds a tier0 test (`TestRecordings_Replay`, `tiergate.Require(t, 0)`) and a tier2 test (`TestRecordings_RealTmux`, `Require(t, 2)`). `PATTERN-test-speed` forbids this: "The functions of one test file name one tier".
  - `impactset.FileTier` takes the lowest tier, so it reads the file as tier0. That file then references `tmuxkit.Socket` and `exec.Command`, both barred in a tier0 file.
  - **Evidence:**
    - `pattern/PATTERN-test-speed.md` (one-tier-per-file rule; tier0 spawn bar)
    - `cmd/lym/tiertags_test.go:115,121-125` (the guard is `//lym:only parent-align finalize`, so no card gate ran it)
    - `internal/impactset/guards.go:225-235` (lowest tier wins)
    - `recordings_test.go` diff lines 389-391 and 434-445
  - **Fix:** move the tier2 test, `realTmux` and the `tmuxkit`/`exec` uses into their own tier2 test file, leaving the shared scenarios in a support file or the tier0 file.
- **F2 — LOW, consistency.** `docs/overview.md:2096-2105`
  - **Summary:** the testkit section describes each kit in turn but leaves out the new `tmuxrec` kit, the repository's only recorded stand-in for tmux, the process table and the user manager below reed.
  - **Evidence:** `docs/overview.md:2092-2105`.
- **F3 — MEDIUM, design.** `internal/testkit/tmuxrec/session.go` (`PlaySession` cleanup)
  - **Summary:** the kit asserts beyond `t.Fatalf` on its own setup. Its cleanup calls `t.Errorf` for unanswered calls. `PATTERN-test-isolation`'s Enforcement section allows that only for the exceptions it names (`shedfake`, `envelope`, `scankit`), and the diff neither adds `tmuxrec` to them nor moves the check into the tests.
  - **Evidence:**
    - `pattern/PATTERN-test-isolation.md`, Enforcement section ("A kit asserts nothing beyond `t.Fatalf` on its own setup, with two exceptions…")
    - `session.go` diff lines 635-639

Tool calls:
1. Searched Go, md and yaml files for stale `sessionLiveness`, `sessionRegistrar`, `paneProcessAge`, `loomcli: strand` and `TestPRSessionSeams` references; searched docs and patterns for `SessionLive`/`RegisterSession`; read the kit entries of `enforcement_test.go`. My context held no repository-wide search for these names or the kit import enforcement list.
2. Searched `docs/research/no-fake-other-modules.md` for landingshed, `shuttlefake.Reed` and envkit rows. My context held no read of the standing-fakes inventory.
