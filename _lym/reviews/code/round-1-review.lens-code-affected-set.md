```yaml
lens: code-affected-set
outcome: reported
```

# Code-affected-set lens: per change, what it reaches and whether it is covered

- **`proc.TreeCPU`/`ProcessCPU` renamed to `TreeSample`/`ProcessSample`, `cpu.go` replaced by `sample.go`.** Covered: no `TreeCPU`, `ProcessCPU` or `cpu.go` reference is left in Go, md or yaml outside `_lym`. Callers `watchdog.go` and `watchdog_test.go`, `proc_linux_test.go`, `proc/doc.go` and `gatecli/doc.go` are all updated. Linux and Windows vet pass.
- **`ReadProcess` added; `ListProcesses` now calls it.** Covered: `proc/doc.go` names it, and the Windows stub is there.
- **`StartFunc`, `RunMemoryCappedVia` and the `runMemoryCappedPlatform`/`runMemoryCapped` signatures.** Covered: every caller is updated, including `export_linux_test.go`, `memorycap.go` and `memorycap_windows.go`. `proc/doc.go` is updated. `testcover.runTestBinary` keeps calling `RunMemoryCapped`, whose signature did not change.
- **`runTiedChild` signature (memory limit and capKilled).** Covered: both production callers (`build.go:54`, `tierrun.go:378`) are updated, and no test calls it.
- **`runInvocations` returning `runStops`, and `tierResult.memoryStopped`.** Covered: `runTier` and `failedOnShortage` are updated. `runnerCrash` is not touched; that is a correctness question outside this lens.
- **`testRequest.memoryQuit`/`memoryGuarding`, and the new lines on stderr.** Not covered: the `stderr` field comment at `cli.go:160` is left stale (finding 2).
- **New hub key `memory_quit_mb`.** Covered in `config.go` (struct, file, cross-pair spec, validation, `MemoryQuit`), `template.yaml`, `gateslot/doc.go:93`, `config_test.go` `templateDefaults` and its rows, and `docs/overview.md`. Other packages that use `busy_quit_sec` (loomshed and landingshed `verifygate`, configcli tests) build their halt way forward from the watchdog line. The guard's kill line carries its own way forward, so they need no change.
- **New tier key `memory_mb`.** Covered in `tiers.go` (`tierFile`, `decodeTier`, `MemoryLimit`), the `template.yaml` key list, the refusal list in `gateslot/doc.go` and `config_test.go` rows.
- **New memory guard section in `gatecli/doc.go`.** Not covered: inserting it moved a CPU-watchdog sentence so it now describes the wrong mechanism (finding 1).
- **New unbounded wait in `memoryguard.go`.** Covered: the `PATTERN-daemon-wakeups.md` list and the `cmd/lym/daemonwakeups_test.go` file list both name it. The directive declares no PATTERN changes, so there is nothing more to land.
- **Help text of `redundant.go` and `testmap.go`.** Covered: no test pins the old Long wording. The phrases dropped from the help ("counted as queued", "side by side") still stand in the package docs, so nothing outside contradicts the new text.
- **Commit messages.** Covered: there is no `Pattern-Departure:` trailer, and every claim in the four commits matches the diff.

## Findings

1. **LOW, consistency, `internal/gatecli/doc.go:728`.** The sentence "It watches `.test` processes only, so a `vstest` tier is bounded by its `timeout_min` alone." belonged at the end of the CPU watchdog section. The new Memory guard section was inserted ahead of it, so the sentence now follows the `memory_mb` paragraph. There "It" reads as the tier's kernel cap, which holds every process of the command, `vstest` included. As a result the doc says a `vstest` tier with `memory_mb` is bounded by `timeout_min` alone, which is wrong. Evidence: `internal/gatecli/doc.go:724-728` (diff hunk `@@ -703,6 +703,28 @@`).
2. **LOW, consistency, `internal/gatecli/cli.go:160`.** The `stderr` field comment lists the lines it receives: "the CPU watchdog's quit, a tier's timeout and each tier's summary". The diff now prints two more lines there: each memory guard kill line (`memoryguard.go` `killTree` → `printLine(g.out, …)` with `out = request.stderr`, `tierrun.go:390`) and the tier's `memory_mb` cap line (`tierrun.go:126-128`). The list is incomplete. Evidence: `internal/gatecli/cli.go:160`, `internal/gatecli/tierrun.go:126-128,390`.

Tool calls:
1. A grep for every file using `busy_quit_sec`/`BusyQuitSec`, plus stale `TreeCPU`/`ProcessCPU`/`cpu.go` references. My context lacked the full set of places that list or handle gate hub keys.
2. A grep of loomshed and landingshed `verifygate`, configcli tests, `watchdogline.go` and `test_integration_test.go` for `busy_quit` handling. I needed to see whether other packages route the watchdog line and would need a matching route for the guard line.
3. A grep for the help phrases the diff dropped, `cli.go` watchdog/stderr comments, tier-key lists and test callers of `runTiedChild`. My context lacked the readers of the changed help text and of the `testRequest` fields.
4. A grep of `gateslot/doc.go`, `docs/overview.md` and configcli docs for lists of cross-pair keys. I needed to check whether a doc enumerates the hub keys that `memory_quit_mb` joins.
