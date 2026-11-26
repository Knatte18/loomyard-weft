---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: tools/machinecost/probe.go:20
    summary: The folder prefix machinecost- is a literal repeated at three creation sites and restated by the test glob, /opt/traps is restated in renderMachine, and the lym_help start name is written twice, against PATTERN-code-conventions' one-named-constant rule.
    fix: "Declare one constant for the folder prefix (for example `probeFolderPrefix = \"machinecost-\"`) and pass it at probe.go:20 (lymtemp.MkdirTemp), fileops.go:21 and newbinary.go:39 (os.MkdirTemp); build probe_test.go:58's glob as `probeFolderPrefix + \"*\"`. In report.go:201 print `trapsPath` through a %s verb instead of the literal /opt/traps. In probe.go declare the lym start name once (for example `lymStartName = \"lym_help\"`) and use it both in measureStarts' name argument and in the Skipped text, which becomes `\"start/\" + lymStartName + \": no lym on PATH and no -lym flag\"`."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: tools/machinecost/main.go:77
    summary: An interrupt or kill during a probe skips every defer and lymtemp.Remove, so the machinecost-* folders inside the -dir folders stay behind with nothing to sweep them, against the entry's and the package doc's claim that the probe deletes every folder it makes.
    fix: "In runProbeCommand (main.go), derive a context with signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM) and pass it to runProbe; have runProbe and each measure function check ctx.Err() between iterations and start their children with exec.CommandContext, so an interrupt returns an error through the normal path and the existing defers remove every folder, then main's lymtemp.Remove runs. State in main.go's package doc that a SIGKILL still leaves the -dir folders behind."
  - id: F3
    severity: MEDIUM
    class: decision
    origin: lens:code-scope
    location: tools/machinecost/main.go:1
    summary: The board entry's Note asks for a measurement report per machine under docs/benchmarks/ beside gate-test-xdr.md; the diff adds none and the record holds no Added-after-Discussion entry deferring it, while the change description names the omission as needing the operator's runs on both machines.
    fix: Leave the code in place and defer this finding to the run's parent. The parent either records the deferral of the docs/benchmarks/ reports with lym loom decision add, or has the operator supply the two machines' reports for this diff. A fixer changes nothing for it.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: tools/machinecost/cli_test.go:62
    summary: The ProbeDirNameGivenTwice row passes even with the -dir flag unregistered, since an undefined flag also exits with exitUsage; the mutation survivor at main.go:108 confirms no test sees a dropped -dir registration.
    fix: "In TestRun_ExitCodes add a row in which a valid -dir gets past parsing and the run then stops at a later refusal with no spawn, such as `probe -dir a=/x -starts 0`, and assert on stderr which refusal fired: capture stderr in a bytes.Buffer for the probe rows and check that the ProbeDirNameGivenTwice row's stderr holds `folder name \"a\" given twice` and the new row's holds `every count must be at least 1`."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: tools/machinecost/probe_test.go:16
    summary: The cleanup on a failed probe, which runProbe's doc and the change description claim, has no test; TestRunProbe_MeasuresEveryPartAndLeavesNoFolders checks for leftovers only after a successful run.
    fix: "Add a tier1 test in probe_test.go that runs runProbe with counts of one and a Dirs list of one valid t.TempDir() folder followed by a NamedDir whose Path is a regular file, so measureNewBinaries fails at os.MkdirTemp after the first folder's measurements and the temp folder exist. Assert runProbe returns an error, and that no machinecost-* entry is left in the valid folder or in filepath.Dir of the temp folder; read the temp folder's parent through lymtemp.Dir() since the failed report carries no TempDir."
  - id: F6
    severity: LOW
    class: design
    origin: handler
    location: tools/machinecost/newbinary.go:80
    summary: go build gets the inherited environment, so its go-build* work folder lands in the system temp folder outside the -dir folders and the probe's own temp folder, which the entry rules out.
    fix: "In linkProbeProgram, add `GOTMPDIR=<the probe's temp folder>` to build.Env, passing tempDir down from measureNewBinaries, so go build makes its work folder inside the probe's temp folder."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: tools/machinecost/newbinary.go:71
    summary: "The doc comment claims the build ignores the caller's GOFLAGS so every machine links the same way, but an empty GOFLAGS= falls back to a go env -w GOFLAGS setting, and CGO_ENABLED, GOAMD64, GOEXPERIMENT and GOTOOLCHAIN still pass through from the caller."
    fix: "In linkProbeProgram, set build.Env so the build is pinned: add GOENV=off (so no go env -w file is read), GOFLAGS= , CGO_ENABLED=0, GOTOOLCHAIN=local and GOEXPERIMENT= besides GOWORK=off. Then reword the doc comment to name what it pins: no workspace, no go env file, no GOFLAGS, no cgo and the local toolchain."
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: tools/machinecost/gitload.go:52
    summary: The git workload's environment passes on the caller's GIT_* variables, so a caller's GIT_TRACE2_EVENT traces the timed rounds, GIT_CONFIG_PARAMETERS or GIT_CONFIG_COUNT bring machine settings back in, and GIT_DIR, GIT_WORK_TREE or GIT_INDEX_FILE point the steps at another repository.
    fix: "In measureGitWorkload, build baseEnv from os.Environ() with every entry whose key starts with GIT_ removed, then append GIT_CONFIG_NOSYSTEM=1 and GIT_CONFIG_GLOBAL=<configPath>. Keep main.go's package doc sentence about the machine's git config, which then holds."
  - id: F9
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: tools/machinecost/main.go:193
    summary: dirList.Set splits a bare path holding = into a name and a relative path, so -dir /tmp/a=b measures ./b under the current directory with no error; a bare path also becomes a name holding /, breaking the slash-separated measurement name shape, and no row pins the relative-to-absolute resolution.
    fix: "In dirList.Set, refuse a name that holds / or is empty with an error naming the name=path form, so a bare path must be given as name=path; keep the split at the first =. Update the package doc's '-dir takes name=path, or a path that then names itself' and the flag usage to match. In TestDirList_Set replace the BarePathNamesItself row with a row expecting the refusal, and add a row with a relative path (for example `a=rel`) asserting Path equals filepath.Join of the test's cwd and rel."
  - id: F10
    severity: LOW
    class: design
    origin: lens:code-tests
    location: tools/machinecost/newbinary.go:71
    summary: Nothing pins that each linked program gets a fresh build ID, the property the new-binary measurement rests on; dropping the -B ldflag or reusing one ID passes every test.
    fix: "Add a tier1 test in a new newbinary_test.go that calls linkProbeProgram twice on a source folder written as measureNewBinaries writes it, reads both binaries, and asserts their contents differ."
  - id: F11
    severity: LOW
    class: design
    origin: lens:code-tests
    location: tools/machinecost/machine.go:172
    summary: readThrottleCount's sum across cores and its nil on an unparsable count have no test, since it reads a fixed /sys glob.
    fix: "Split readThrottleCount into the glob read and a pure sumThrottleCounts(texts []string) *float64; readThrottleCount passes each file's text to it. Add a tier0 TestSumThrottleCounts in report_test.go with rows for two counts summed, an unparsable count giving nil, and no counts giving nil."
  - id: F12
    severity: NIT
    class: design
    origin: lens:code-correctness
    location: tools/machinecost/probe.go:52
    summary: A probe with no -dir writes no new-binary or file-operation figures yet records nothing under Skipped, while the lym starts it leaves out are recorded there.
    fix: "In runProbe, when config.Dirs is empty, append `newbinary, fileops: no -dir folder given` to report.Skipped, and add the expected Skipped entry to a probe test whose config has no Dirs, or note the case in TestRunProbe's wantSkipped by giving one run no Dirs."
---

### [BLOCKING] Repeated literals for the folder prefix, the traps path and the lym start name

**Location:** `tools/machinecost/probe.go:20`, `tools/machinecost/fileops.go:21`, `tools/machinecost/newbinary.go:39`, `tools/machinecost/probe_test.go:58`, `tools/machinecost/report.go:201`, `tools/machinecost/probe.go:47-48`

**Issue:** `PATTERN-code-conventions` requires a name or path that can go stale to be one named constant, declared once and referenced everywhere.
The `machinecost-` prefix is written at three creation sites, and the leftover check in the probe test restates it, so a rename at one site lets that site's leftovers slip past the check.
`renderMachine` prints `/opt/traps` as a literal beside `trapsPath`, and `lym_help` is written twice in `runProbe`.
Every violation of a listed constraint is BLOCKING under this round's rules.

### [MEDIUM] An interrupted probe leaves its folders in the -dir folders

**Location:** `tools/machinecost/main.go:77`

**Issue:** The board entry says the probe deletes every folder it makes, and the package doc says it deletes them before it exits.
Go's default SIGINT and SIGTERM handling ends the process without running `defer`s or `lymtemp.Remove`.
The lymtemp run folder is swept by a later lym start, but the `machinecost-*` folders inside `/tmp/lym-*` and `/tmp/zz-*` have no sweeper.
A probe with the default counts runs long enough that an operator pressing Ctrl-C is the ordinary case.

### [MEDIUM] The docs/benchmarks/ measurement reports are left out without a recorded deferral

**Location:** `tools/machinecost/main.go:1`

**Issue:** The entry's Notes say the result goes into a measurement report under `docs/benchmarks/` for each machine.
The diff holds only `tools/machinecost/`, and the decision record has no `Added after Discussion` entry.
`_lym/landing/summary.md` says the reports need the operator's runs on both machines.
That reason is sound, but the choice to drop part of the entry is the parent's to record, not the fixer's to make.

### [MEDIUM] ProbeDirNameGivenTwice cannot tell a refused duplicate from an unknown flag

**Location:** `tools/machinecost/cli_test.go:62`

**Issue:** With `flags.Var(&dirs, "dir", ...)` dropped, `-dir` becomes an undefined flag, and parsing still returns `exitUsage`.
The mutation report lists the drop at `main.go:108` as a survivor.
No test shows that a valid `-dir` is accepted by the command line at all.

### [MEDIUM] Cleanup after a failed probe is claimed but untested

**Location:** `tools/machinecost/probe_test.go:16`

**Issue:** `runProbe`'s doc says it deletes its temp folder and every folder it made, on failure too, and the change description repeats it.
The one probe test checks for leftovers only after a run that succeeds.

### [LOW] go build's work folder lands in the system temp folder

**Location:** `tools/machinecost/newbinary.go:80`

**Issue:** `build.Env` is the caller's environment plus `GOWORK` and `GOFLAGS`.
Under an operator's shell, `go build` makes its `go-build*` work folder in the system temp folder, outside the `-dir` folders and the probe's own temp folder.
The entry rules that out, and the package doc's last paragraph claims it.

### [LOW] linkProbeProgram's environment does not pin the build its doc says it pins

**Location:** `tools/machinecost/newbinary.go:71`

**Issue:** The go command treats an empty `GOFLAGS` as unset and reads `go env -w` settings from its env file.
`CGO_ENABLED`, `GOAMD64`, `GOEXPERIMENT` and `GOTOOLCHAIN` reach the build from the caller.
Two machines can therefore link different programs, against the doc comment's "every machine links it the same way".

### [LOW] Inherited GIT_* variables reach the git workload

**Location:** `tools/machinecost/gitload.go:52`

**Issue:** `baseEnv` starts from `os.Environ()`.
The package doc says the workload ignores the machine's git config, and `measureGitWorkload`'s doc keeps the trace out of the timed rounds.
A caller's `GIT_CONFIG_PARAMETERS`, `GIT_CONFIG_COUNT` or `GIT_TRACE2_EVENT` breaks both claims, and `GIT_DIR` or `GIT_WORK_TREE` sends the steps elsewhere.

### [LOW] dirList.Set mis-splits bare paths and builds names holding /

**Location:** `tools/machinecost/main.go:193`

**Issue:** `strings.Cut(value, "=")` treats any `=` as the separator, so a bare path holding one becomes a name and a relative path, and the wrong folder is measured silently.
A bare absolute path becomes its own name, giving keys like `fileops//tmp/zz-probe/stat_us`, against the slash-separated key shape `Measurement`'s doc states.
These keys pair across machines only when the full paths match.
No row covers resolving a relative path.

### [LOW] No test pins a fresh build ID per linked program

**Location:** `tools/machinecost/newbinary.go:71`

**Issue:** The new-binary measurement exists to time a hash the endpoint agent has never seen.
The probe test pins only the measurement names, so a build without `-B`, or with one reused ID, passes.

### [LOW] readThrottleCount is untestable as written

**Location:** `tools/machinecost/machine.go:172`

**Issue:** The other `/proc` readers split parsing from reading and have tier0 rows.
`readThrottleCount` reads a fixed `/sys` glob, so its sum and its nil-on-unparsable rule have no test.

### [NIT] A probe with no -dir does not say it skipped the per-folder parts

**Location:** `tools/machinecost/probe.go:52`

**Issue:** The report records the left-out lym starts under `Skipped` but stays silent when no `-dir` is given and the new-binary and file-operation parts do not run.

## Rejected

- code-tests finding 4 (traced round kept out of the timings untested): the separation is structural; with a timing figure no test can tell the traced round from a timed one without comparing real times, which `PATTERN-test-speed` forbids.
- code-tests finding 5 (different-counts warning untested): the warning is text only a human reads, and the testing rules give such output no test; nothing reads it.
- `PATTERN-gitrepo-client-boundary` for the raw `git` starts: the workload measures bare git process cost on throwaway repositories, writes no lym remote-tracking ref, and the checked-call scan covers `internal` and `cmd` only, with `tools/deploy` already running raw git; no finding.
- `PATTERN-child-processes` spawn logging: the background puts dev tooling under `tools/` outside the rule.
- The probe test's leftover glob over the shared run folder: only this test creates `machinecost-*` entries in the test binary's run folder, so nothing else can affect it.
