```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each function and branch the diff adds through a concrete case, including empty, zero and boundary inputs and a failure of each call it makes.

**Per function or branch: the case I ran, and whether the code is correct**

- `main`: `compare` creates no run directory, so `lymtemp.Remove` returns nil and the exit code is unchanged. Correct. With SIGINT or SIGTERM during a probe, Go's default handler exits without running any `defer` and without `lymtemp.Remove`. Not correct (F1).
- `run`: with no arguments it returns `exitUsage`; with `measure` it returns `exitUsage`. Correct.
- `runProbeCommand`:
  - `-starts 0` returns `exitUsage`. Correct.
  - `-h` returns `flag.ErrHelp`, so the exit code is 2. That is acceptable.
  - With `lym` absent from PATH, `LookPath` fails and the lym starts are recorded under skipped. Correct.
  - With no `-dir`, the new-binary and file-operation parts run zero times and nothing is recorded under skipped. The report gives no sign that those parts were left out (F5).
  - When `writeReport` fails, the whole run's figures are lost and only an error is printed. That is acceptable for a tool.
- `runCompareCommand`: with one argument it returns `exitUsage`; with a missing file it returns `exitError`. Correct.
- `dirList.Set`:
  - `a=/x` is correct, and `a=` / `=/x` are refused. Correct.
  - A bare `/tmp/zz-probe` gives the name `/tmp/zz-probe`, so the key becomes `newbinary//tmp/zz-probe/first_run_ms`. The doubled slash makes the slash-separated name ambiguous, but the key still pairs across machines when the path is the same.
  - A bare path that contains `=`, such as `/tmp/a=b`, gives the name `/tmp/a` and the relative path `b`, which resolves under the current directory. The wrong folder is measured without any error (F4).
- `dirList.String`: with an empty list it returns `""`. Correct.
- `runProbe`:
  - When `readMachine` fails, the deferred `RemoveAll(tempDir)` runs. Correct.
  - When a measurement fails, every folder made so far is removed by that part's own `defer`. Correct.
  - On a signal, nothing is removed (F1).
- `measureStarts`: with count=1 and `/bin/true`, the median is the single sample, and the user and system times are means per start. The wait4 rusage includes children the started process waited on. Correct.
- `measureNewBinaries`:
  - With two folders, `sourceDir` is reused through `MkdirAll`. Correct.
  - If `-dir` does not exist, `MkdirTemp` returns a clear error. Correct.
  - If the build fails, `RemoveAll(binaryDir)` runs. Correct.
- `linkProbeProgram`:
  - With `go env -w GOFLAGS=...` set, `GOFLAGS=` does not clear it. The go command reads an empty variable as unset and falls back to the go env file, so the doc comment's claim is false.
  - `CGO_ENABLED`, `GOAMD64`, `GOEXPERIMENT` and `GOTOOLCHAIN` are inherited from the caller (F2).
  - `go build` also makes its `go-build*` work folder in the system temp folder, because `TMPDIR` is inherited, and it writes to GOCACHE. Both are folders outside the `-dir` folders and the probe's temp folder (F1).
- `timeRun`: when the program exits non-zero, the error names the binary. Correct.
- `measureFileOps`:
  - With count=1, the defers run in the order Close, then `RemoveAll`. Correct.
  - When stat fails, the error carries `workDir` and the folder is still removed. Correct.
- `measureGitWorkload`:
  - With rounds=1, `traceDir` is made once and each step's trace is read. Correct.
  - If the caller's environment already holds `GIT_TRACE2_EVENT`, `GIT_DIR`, `GIT_CONFIG_PARAMETERS`/`GIT_CONFIG_COUNT`, `GIT_INDEX_FILE` or `GIT_WORK_TREE`, `baseEnv` passes them on. The timed rounds then trace, or take injected config, or go to another repository (F3).
- `runGitRound`:
  - With global config replaced and no `init.defaultBranch`, git uses its built-in default, and `push origin HEAD` pushes whatever branch that is. Correct.
  - When a step fails, the error carries git's output and `roundDir` is removed. Correct.
- `runGit`: `CombinedOutput` is followed by `ProcessState` user and system time. Correct.
- `readMachine`: on ARM, `/proc/cpuinfo` has no "model name" and the model is left `""`. Acceptable. When `go` is missing, it returns an error. Correct.
- `readSnapshot` and `readOptionalFile`: when a file is missing, they return `""` and the field is left nil. Correct.
- `commandOutput`: when the command fails, the error names the command. Correct.
- `parseLoadavg`: `""` gives nil, nil, nil, and the kernel line parses. Correct.
- `parseCPUPressure`: a text with no `some` line gives nil, nil. Correct.
- `parseCPUInfo`: with no `cpu MHz` line, `meanMHz` is nil. Correct.
- `readThrottleCount`: with no thermal files it returns nil. If one core's file cannot be parsed, the whole count is nil. That is acceptable.
- `parseOptionalFloat`: `"x"` gives nil. Correct.
- `median`: an even count of 4 gives the mean of the two middle samples, using integer division of the duration. Correct.
- `writeReport` and `readReport`:
  - A round trip returns the same report, and `json.Unmarshal` accepts unknown keys. Correct.
  - A file that is not a report gives an error that names the path. Correct.
- `compareMeasurements`:
  - A base of 0 gives a nil ratio, and a name missing from one side gives a nil on that side. Correct.
  - `&m.Value` takes the address of a per-iteration loop variable, which is safe since `go.mod` declares go 1.26. Correct.
- `renderComparison`:
  - Different counts trigger the warning. Correct.
  - Different `Dirs` or `LymBinary` produce no warning; the rows are only one-sided. That is acceptable.
- `formatOptional`: nil prints `n/a`. Correct.

**Findings**

- **F1 — MEDIUM, design.** Locations: `tools/machinecost/main.go:611-618`, `probe.go:870-875`, `newbinary.go:830-832`.
  An interrupted or killed probe leaves `machinecost-*` folders in the `-dir` folders. Nothing sweeps those folders: only the lymtemp run directory is swept on the next start. The `go build` child also makes its `go-build*` work folder in the inherited system temp folder.
  Both contradict the package doc's claim at `main.go:567-568` that the probe "creates folders only inside each -dir folder and in its own temp folder ... deletes every one of them before it exits". They also contradict the entry's "makes none outside the `--dir` folders and its own temp folder".
  Evidence:
  - `main.go:611-618` registers no signal handling, and cleanup runs only through `defer` and `lymtemp.Remove`.
  - `newbinary.go:832` sets `build.Env` to `os.Environ()` plus `GOWORK` and `GOFLAGS`, with no `TMPDIR`.
- **F2 — LOW, design.** Location: `tools/machinecost/newbinary.go:822,832`.
  The claim "The build ignores ... the caller's GOFLAGS, so every machine links it the same way" is false in two ways:
  - `GOFLAGS=` set to an empty value falls back to a `go env -w GOFLAGS` setting.
  - `CGO_ENABLED`, `GOAMD64`, `GOEXPERIMENT` and `GOTOOLCHAIN` are inherited from the caller.

  Evidence: `newbinary.go:832`, `build.Env = append(os.Environ(), "GOWORK=off", "GOFLAGS=")`.
- **F3 — LOW, design.** Location: `tools/machinecost/gitload.go:227`.
  The workload's environment inherits the caller's `GIT_*` variables. The results change in three ways:
  - A `GIT_TRACE2_EVENT` set by the caller makes every timed round trace. That is the cost the doc at `gitload.go:220-221` says the timed rounds are kept free of.
  - `GIT_CONFIG_PARAMETERS`/`GIT_CONFIG_COUNT` bring the machine's git settings back in, against the doc at `main.go:560-561`.
  - `GIT_DIR`/`GIT_WORK_TREE`/`GIT_INDEX_FILE` send `add`, `commit` and `push` to another repository.

  Evidence: `gitload.go:227`, `baseEnv := append(os.Environ(), "GIT_CONFIG_NOSYSTEM=1", "GIT_CONFIG_GLOBAL="+configPath)`.
- **F4 — LOW, design.** Location: `tools/machinecost/main.go:727-745`.
  A bare `-dir` path that contains `=` is split into a name and a relative path. The probe then measures a folder under the current directory instead of the one given, and reports no error.
  Bare paths also produce names that contain `/`, such as `fileops//tmp/zz-probe/stat_us`. That breaks the slash-separated key shape the doc of `Measurement` at `report.go:46` states.
  Evidence: `main.go:728-731`, which uses `strings.Cut(value, "=")` and falls back to `name, path = value, value` only when there is no `=`.
- **F5 — NIT, design.** Location: `tools/machinecost/main.go:641,656-661`, `probe.go:903`.
  A probe run with no `-dir` writes no new-binary or file-operation figures and records nothing under `Skipped`. Lym starts that are left out are recorded there.
  Evidence: `probe.go:897-903`.

**Tool calls**

1. `grep go.mod`, `cmd/lym/main.go` lymtemp and signal lines, and `go env GOFLAGS GOTOOLCHAIN`: I needed the module's Go version to judge loop-variable capture, and the environment's GOFLAGS and GOTOOLCHAIN to judge F2. The manifest held neither.
