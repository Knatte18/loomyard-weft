```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each thing the diff changes, I looked for what calls, reads or asserts it, and checked the repo's rules against the change.

**Per change, the set found, covered or not**

1. **New package `tools/machinecost`: a `main` package, its CLI and its JSON report.**
   - Callers or readers outside the package: none. There is no tools index doc, and only `docs/benchmarks/*` name other tools.
   - Covered: nothing outside is left stale.
2. **New process starts with `os/exec`: `go build`, `git`, `/bin/true` and the lym binary.**
   - `PATTERN-child-processes`: tools sit outside its spawn-logging rule (background "dev tooling under `tools/` … outside this rule"). Each start waits on its child through `Run`/`CombinedOutput`/`Output`.
   - `cmd/lym/spawnobservability_test.go:46`: the walk never visits `tools/`.
   - Covered.
3. **New raw git with `exec.Command(gitBinary, …)` in `gitload.go`.**
   - `PATTERN-gitrepo-client-boundary`'s checked-call scan only walks `internal` and `cmd` (`cmd/lym/checkedcall_test.go:50`). `tools/deploy/main.go:175,377` already runs raw git.
   - The workload runs against its own throwaway repositories, writes no remote-tracking ref of a lym repository and takes no lock.
   - Covered; no finding.
4. **New temp folders through `lymtemp.MkdirTemp`, and `os.MkdirTemp(dir.Path, …)` in the `-dir` folders.**
   - `PATTERN-lym-files` temp-root rule: the run directory comes from lymtemp, `main` calls `lymtemp.Remove`, and the `-dir` calls name a directory that is neither `""` nor `os.TempDir()`, which the bare-temp scan allows.
   - Covered.
5. **New test files: `cli_test.go` and `report_test.go` (tier0), `probe_test.go` (tier1), `testmain_test.go`.**
   - `PATTERN-test-speed` tier0 purity: the tier0 files have no `exec.Command` and no gitkit spawn.
   - `PATTERN-test-isolation`: `TestMain` calls `gitkit.HermeticGitEnv()` and `tmuxkit.Main`.
   - Each test opens with `tiergate.Require`, and `go-cmp` is already in `go.mod:12`.
   - Covered.
6. **Functions reachable only from tests (`PATTERN-test-only-code`).**
   - The scan covers `./tools/...` (`cmd/lym/testonlycode_linux_test.go:30`). `ptr` is in a `_test.go` file, and every production function is reached from `main`.
   - Covered.
7. **Names that can go stale (`PATTERN-code-conventions`, Named constants).**
   - The `machinecost-` folder prefix is a repeated literal: `probe.go:20`, `fileops.go:21`, `newbinary.go:39`, plus the test glob at `probe_test.go:58`.
   - `/opt/traps` is restated at `report.go:201` beside `trapsPath` at `machine.go:26`.
   - The `lym_help` start name appears twice, at `probe.go:47` and `probe.go:48`.
   - **Not covered** (finding 1).
8. **Commit messages and departures.**
   - No `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailer is present, and none is needed: the change is new code, no fix and no loosened test.
   - The messages claim no agreement or improvement figures.
   - Covered.
9. **Declared PATTERN changes.**
   - None are declared in the directive.
   - Covered.
10. **Stencil wording.**
    - The diff changes no stencil.
    - Not applicable.

**Findings**

1. **BLOCKING · class `design` · `tools/machinecost/probe.go:20`**
   - Summary: the probe's folder prefix `"machinecost-"` is repeated as a literal at three creation sites, and the cleanup test's glob `"machinecost-*"` restates it. This breaks PATTERN-code-conventions' "one named constant, declared once and referenced everywhere".
   - Why it matters: a rename at one creation site lets that site's leftovers slip past the test's leftover check without anything failing.
   - The same gap shows twice more: `report.go:201` restates `/opt/traps` instead of using `trapsPath`, and `"lym_help"` appears twice at `probe.go:47-48`.
   - Fix: declare one prefix constant, use it at all three `MkdirTemp` sites, and build the test glob from it. Render `trapsPath` in `renderMachine`, and name the lym start once.
   - Evidence:
     - `tools/machinecost/probe.go:20`
     - `tools/machinecost/fileops.go:21`
     - `tools/machinecost/newbinary.go:39`
     - `tools/machinecost/probe_test.go:58`
     - `tools/machinecost/report.go:201`
     - `tools/machinecost/machine.go:26`
     - `tools/machinecost/probe.go:47-48`
     - `pattern/PATTERN-code-conventions.md:11`

**Tool calls**

1. Grepped for the `gitexec:raw` scan's roots, for docs that name tools, and for `tools` in the gate config. My context did not show whether the checked-call scan or any doc index covers `tools/`.
2. Grepped `go.mod` for go-cmp, looked for `gate.yaml`, and grepped the package for the repeated literals. I needed the exact line numbers of each repeated literal and to confirm the test dependency is already declared.
