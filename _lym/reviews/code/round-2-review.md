---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    carries: F2
    location: tools/machinecost/newbinary.go:84
    summary: "An interrupt during go build ends the go command at once, since go dies on SIGINT, so the probe deletes its temp folder while go's compiler or linker is still running and writing into it, and that process outlives the probe. The comment above build.Cancel says the opposite."
    fix: "In linkProbeProgram (tools/machinecost/newbinary.go), start the build with exec.Command(goBinary, ...) instead of exec.CommandContext. Delete the build.Cancel line and the two comment lines above it. After build.CombinedOutput() returns without error, return ctx.Err() when it is non-nil, so an interrupted probe stops once the build is done (well under a second with a warm cache). Signals sent to machinecost alone, such as SIGTERM or SIGHUP, then never interrupt a build. A terminal interrupt reaches the whole process group, so go and its compiler and linker end together. measureNewBinaries keeps passing ctx to timeRun unchanged."
  - id: F2
    severity: LOW
    class: design
    carries: F6
    location: tools/machinecost/newbinary.go:87
    summary: "go build still writes outside the -dir folders and the probe's temp folder. It writes build cache entries under the user's GOCACHE (15 new files in a warm cache for 3 links, 229 in a cold one) and telemetry counters under the user config folder's go/telemetry, creating both folders on a machine that lacks them. The board entry and the package doc say the probe makes no folder outside those places."
    fix: "In linkProbeProgram's build.Env (tools/machinecost/newbinary.go), add GOCACHE=<tempDir>/gocache and XDG_CONFIG_HOME=<tempDir>/config, each built with filepath.Join(tempDir, ...). Declare the two folder names as named constants beside probeProgramSource. Extend the doc comment's last sentence to say that the go command's work folder, build cache and telemetry are all made inside tempDir. The first link of a probe then compiles the runtime from cold, which happens outside the timed runs."
prior:
  - id: F1
    status: resolved
  - id: F2
    status: open
    answer: "The -dir folders are now removed on interrupt, SIGTERM and hangup, but an interrupt during go build still leaves go's linker running after the probe exits. Carried as F1."
  - id: F3
    status: resolved
    answer: "Settled by the added entry \"The docs/benchmarks/ measurement reports follow the operator's runs, outside this change\"."
  - id: F4
    status: resolved
  - id: F5
    status: resolved
  - id: F6
    status: open
    answer: "The go-build work folder now lands in the probe's temp folder, but go build still writes its build cache and telemetry outside it. Carried as F2."
  - id: F7
    status: resolved
  - id: F8
    status: resolved
  - id: F9
    status: resolved
  - id: F10
    status: resolved
  - id: F11
    status: resolved
  - id: F12
    status: resolved
---

I built the tool with `go build -o` into the session scratchpad and ran it against two scratch folders. A run with counts of two exited 0 and left both folders empty.

### [MEDIUM] An interrupted go build leaves the linker running after the probe exits

**Location:** `tools/machinecost/newbinary.go:84`

**Issue:** `build.Cancel` sends `os.Interrupt` to the go command.
The comment above it says that an interrupted go command waits for its compiler and linker.
It does not.
I sent SIGTERM to machinecost alone (`timeout --foreground -s TERM 1.2`) during the new-binary part.
The probe reported `build probe program: signal: interrupt`, which means go was killed by the signal rather than exiting on its own.
After the probe had exited, `pgrep` still showed `link -o /tmp/lym-1000/rjkby/machinecost-3814125172/go-build724080963/b001/exe/a.out ... -B=0xd7c7…` running, and its run folder `rjkby` had already been removed.
That linker races the probe's `os.RemoveAll(tempDir)` and `lymtemp.Remove`.
A file it writes during the removal can make the removal fail and leave a folder behind, against the package doc's claim that every folder is deleted on interrupt, SIGTERM or hangup.
The rest of F2 holds: the `-dir` folders were empty after each interrupted run (SIGINT during the new-binary part, SIGINT during the git workload, SIGTERM during the build).

### [LOW] go build still writes its build cache and telemetry outside the probe's folders

**Location:** `tools/machinecost/newbinary.go:87`

**Issue:** `GOTMPDIR` moves the go command's work folder into the probe's temp folder, but the build also writes to the build cache and to Go telemetry.
I gave the probe a scratch `GOCACHE`: one run with `-binaries 2` per folder over two folders left 229 files in it, and a second run with `-binaries 3` added 15 more.
I gave it a scratch `HOME`: the run created `.config/go/telemetry/local/` there, holding `compile`, `asm` and `link` counter files.
With `GOENV=off`, `GOCACHE` falls back to `~/.cache/go-build` even when the user's go env file points elsewhere.
The board entry says the probe makes no folder outside the `--dir` folders and its own temp folder.
The package doc says the probe creates folders only inside each `-dir` folder and its own temp folder.
Neither holds on a machine without those Go folders, and on any machine the probe writes files in folders it does not own.

## Rejected

- Named constants (focus 5): `"probe"` and `"compare"` appear both as dispatch cases and in message prefixes, and `machinecost` appears in the error and progress prefixes. None of those lines changed this round, so an incremental round cannot raise a finding on them. Every other tool under `tools/` writes its program name in its message prefixes the same way. The literals this round touched are now named constants: `reportNameFormat`, `probeFolderPrefix`, `lymStartName`, `trapsPath`, `goBinary` and `gitBinary`. Within the package, `GOWORK=off` and `GOTOOLCHAIN=local` are toolchain settings, not names that can go stale.
- `-dir` contract (focus 3): `dirList.Set` refuses an empty name, an empty path, a name holding `/`, a bare path, and a bare path holding `=`. It keeps a path holding `=` after the name, such as `a=/tmp/x=y`, and resolves a relative path against the current folder. The package doc, the flag usage and the error text agree, and `TestDirList_Set` has a row for each case. No defect.
- Tests (focus 4): each named test fails when its behavior is removed. Dropping the duplicate-name check makes `ProbeDirNameGivenTwice` reach runProbe and exit `exitError`. Moving runProbe's temp-folder removal to the success path leaves a `machinecost-*` folder in the run folder that `TestRunProbe_FailedRunLeavesNoFolders` globs. Dropping `-B` makes the two links byte-identical. Dropping the sum fails the `SumsEveryCore` row. The mutation survivor at `newbinary.go:82` (dropping `build.Dir`) still passes `TestLinkProbeProgram_EachLinkDiffers`, because the test then builds the machinecost package itself. That is a gap in what the test checks, not a regression, and an incremental round raises only BLOCKING findings, so it is not filed. Running the linked binary and checking that it exits 0 would catch it.
- The other mutation survivors drop `usageText`, `renderComparison` or `renderMachine` lines. That text is read only by a person, so no test is owed. The `ParseFloat` bit-size survivor at `machine.go:206` and the `buildIDBytes` survivor at `newbinary.go:75` change nothing a caller relies on.
- Child environments (focus 2): the git workload drops `GIT_*`, ignores the system config and uses the probe's global config. It runs inside the temp folder and writes nothing under `HOME`. The timed starts also drop `GIT_*`, as `measureStarts`' doc says. The go build pins are covered under the second finding above.
- A `-dir` folder that does not exist fails the probe with `create folder in /tmp/lym-probe-nonexistent-xyz: stat ...: no such file or directory`. The message names the folder, so this is no defect.
