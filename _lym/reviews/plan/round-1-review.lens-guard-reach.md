```yaml
lens: guard-reach
outcome: reported
```

Task: guard-reach lens. For each guard, refusal, branch or test row a card adds, decide whether a real entry point can reach it.

**Per guard: reachable or not**
- Card 1, trailer key/value refusal: reachable (direct call).
- Card 3, `CompileKeyPatterns` refusals: reachable through `LoadConfig` (card 4).
- Card 4, `LoadConfig` refusals for `diff` and `command` without `compare`: reachable.
- Card 4, `refuseHubOnlyKeys`: reachable. `PrepareRunSettings` runs it beside `refuseRetiredKeys`, before `checkLocalDocument`, so the generic guard-category refusal does not shadow it (configreg/runsettings.go:70-88).
- Card 4, `FlagDiffMode` flags: reachable.
- Cards 5 and 6, parser refusals: reachable through the `expectation` and `judgement` entries, and for card 5 also through `amend --entries`.
- Card 8, `CheckSeal` branches (`discard`, `Unsealed`, `NotYet`, `Empty`, the `--discard` and `--restore` faults): reachable.
- Card 8, `CheckBaseline` missing-directory fault: **not reachable when no `Drift-Baseline` commit exists** (finding 2).
- Card 9, `CaptureCurrent` slot and lock wait, step refusal as `Passed` false: reachable.
- Card 10, `Check` steps 1–7 and the gate pass/fail rows: reachable, except step 5 `NotYet` (finding 3).
- Card 11, wrap writes a report on fail and nothing on pass: reachable.
- Card 12, Drift-Expect Done, moved-head, missing-baseline, mark-probe and seal branches: reachable. The missing-baseline Stuck depends on card 8's fault (finding 2).
- Card 12, `driftExpectEntry` key refusal: reachable from the recipe config.
- Card 14, check before Drift-Expect: reachable only at a running Drift-Expect (finding 3).
- Card 15, `Changes`: reachable, but it also fires on loom's own weft commits (finding 1).
- Card 16, steps 1–4 and 6–10: reachable.
- Card 16, step 5 compare of a resumed judge against the recorded snapshot: reachable, but can never pass (finding 1).
- Card 16, `driftVerifyEntry` refusal of `gate_key`: reachable.
- Card 16, interim `command`-mode Stuck: reachable on a `diff: command` hub.
- Card 17, stored `JudgedCase` Stuck at the same head: reachable.
- Card 19, `Amend` refusals (running row, other row, before Drift-Expect, none or several flags, restore with nothing failed): reachable.
- Card 19, `Defer` admission at the writer: reachable only through its `CheckBaseline` branch. The writer's `drift` entry passes on baseline faults, so no writer Stuck carries `DeferHaltPrefix`; that matches D12.
- Card 19, `Defer`'s `CaptureHeld` refusal: reachable, but with a wrong way forward (finding 4).
- Card 19, `checkParentCaller`: reachable.
- Card 26, `renderCommand` `{skip_file}` refusal: reachable.

**Findings**

1. **BLOCKING, design.** Cards 15 (`JudgeSnapshot.Changes`) and 16 (step 5, step 10).
   - The snapshot guard counts loom's own weft writes as judge edits. The run status `_lym/shed/&lt;slug&gt;/status.json` is tracked on the weft, and loom commits it at every row transition (`loom: &lt;row&gt; -&gt; running`).
   - Take an interrupted judge, or a Stuck after the session, followed by `lym loom resume`. The resume moves the weft HEAD and changes `status.json` after the snapshot. Step 5 compares "against the recorded snapshot, never a new one, until the trees match", so the resumed row is a judge-change Stuck every time.
   - Its stated way forward, resetting the moved weft branch, would drop loom's own status commits.
   - `Changes` exempts only `judgement.yaml` and gives no exemption for loom-owned commits or run files.
   - Evidence: `git -C _lym ls-files` lists `shed/drift-rework/status.json`; `git -C _lym log` shows `loom: Plan-Burler-Review -&gt; running` and others; `internal/shedrun/paths.go:151` (`StatusFile` under `RunDir`); card 15 line 65; card 16 lines 120 and 126.

2. **MEDIUM, consistency.** Card 8 (`CheckBaseline` / `BaselineState.Unguarded`), used by cards 10, 12 and 16.
   - "Unguarded, true when no commit carries the trailer, which checks nothing" leaves the missing-baseline-directory fault unreachable on a run with no `Drift-Baseline` commit. That is a capture counted before this lands, which only gets its digest on the seal commit.
   - D12 requires a counted capture whose baseline directory is gone to be Stuck at Drift-Expect naming `lym drift defer`. Card 12 reads that through `CheckBaseline`, so in this state Drift-Expect goes on to `ListCaptures` of a missing directory and returns an I/O error, not the defer Stuck.
   - Card 8's test rows never combine a missing directory with no trailer commit.
   - Evidence: card 8 line 49; card 12 line 73; decision record D12, line 306.

3. **MEDIUM, consistency.** Card 10 `Check` step order (step 3 before step 5), and card 14's test.
   - A scope record with no `awaited_at` stops at step 3 with `BaselineFault` ("the capture never counted", naming `lym drift defer`) before step 5's `NotYet`.
   - In a real run, `awaited_at` is unset at Drift-Baseline and during Drift-Await. Drift-Await sets it when it counts (records.go:77), and the row then moves to Drift-Expect.
   - So `lym drift check` before Drift-Expect wrongly tells the caller to defer, against D6 ("before that point `lym drift check` reports that no expectation is sealed yet and checks nothing"). `NotYet` is reachable only while Drift-Expect itself runs.
   - Card 14's row "status at Drift-Await … prints nothing is sealed yet" seeds `awaited_at` at Drift-Await, a state a real run never holds, so the test passes while the real path misreports.
   - Evidence: card 10 lines 49 and 51; card 14 lines 29-30; `internal/driftengine/records.go:77`.

4. **LOW, consistency.** Card 19 (`Defer`) together with card 9 (`CaptureCurrent`).
   - `Defer` still refuses while the capture lock is held, with the way forward "lym loom resume, whose row waits on the running capture".
   - After card 9, any `lym drift check` or `drift` gate entry holds that same lock during its current capture. The refusal now also fires for a check's capture, where resuming the row is not the way forward; waiting for the check to finish is.
   - Card 19 keeps the refusal unchanged and plans no row for a lock held by a current capture.
   - Evidence: `internal/driftengine/verbs.go` `Defer` (`CaptureHeld` refusal text); card 9 line 47; card 19 line 53.

Tool calls:
1. `records.go` `BaselineHead`/`AwaitedAt` and `configreg/runsettings.go` / `shedrun` paths, partial: the manifest lacked the field set, the refusal order and where run files live.
2. `configreg/runsettings.go` `PrepareRunSettings` body and run-dir writers in the seat path: to confirm `refuseHubOnlyKeys` is reachable before the generic local check.
3. `git ls-files _lym` / `git log -- _lym/shed` from the warp: tried to confirm the run status is tracked; it returned nothing because `_lym` belongs to the weft.
4. `git -C _lym ls-files` and `git -C _lym log`: confirmed `status.json` is weft-tracked and that loom commits it at row transitions.
