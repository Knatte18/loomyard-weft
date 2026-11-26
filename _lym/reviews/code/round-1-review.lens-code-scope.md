```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each goal point of the board entry, and the record's Added-after-Discussion entries (there are none), to the diff's changes.

**Per change → decision**
- C1 `main.go`: `run` dispatch, probe/compare flags, `dirList`, the package doc with its `go build -o` usage. Traces to Idea "one machine-readable report per run, and a compare mode", Note 1 (build as a binary) and Note 4 (the `-dir lym=/tmp/lym-*` / `zz=/tmp/zz-*` usage).
- C2 `starts.go`: traces to Idea "Process starts" (`/bin/true`, `git --version`, lym; median µs; children's user and system time).
- C3 `newbinary.go`: traces to Idea "The first run of a binary with a new hash" (`-ldflags=-B=0x<random>`, first and second run per `--dir`). It also traces to the earlier note's "time a freshly built test binary waits".
- C4 `fileops.go`: traces to Idea "File operations in each `--dir`". It also traces to the earlier note's "file creation with and without a scanned folder".
- C5 `gitload.go`: traces to Idea "A git workload shaped like `internal/fabricengine`'s tests" (wall, user and sys per step, plus the processes started). The probe's own git config is forced by the Idea's "runs the same way on every machine".
- C6 `machine.go`: traces to Idea "A machine descriptor" and Note 2 (record the load). Hostname, CPU model, cores, kernel, governor, mean MHz, throttle count, load, CPU pressure before and after, Go and git versions, and `/opt/traps` are all present.
- C7 `report.go`: traces to Idea "One machine-readable report per run, and a compare mode that prints the ratio".
- C8 `probe.go`: the orchestration and its temp folder under `lymtemp`. Traces to Idea "It deletes every folder it makes…". The `Skipped` list and the `-lym` flag are forced by C2's lym start.
- C9 `cli_test.go`, `probe_test.go`, `report_test.go`: forced by C1–C8.
- C10 `testmain_test.go`: forced by PATTERN-test-isolation for the new test package.

**Per decision → changes**
- Idea, process starts: C2, C8
- Idea, new-hash binary first and second run: C3
- Idea, file operations per `--dir`: C4
- Idea, git workload: C5
- Idea, machine descriptor: C6
- Idea, report plus compare mode: C1, C7
- Idea, delete every folder made; none outside the `--dir` folders and its temp folder: C3, C4, C5, C8 (whether this holds on every exit path is for the correctness lens)
- Note 1, build as a binary with `go build -o`: C1 (doc)
- Note 2, record the load: C6, C7
- Note 3, measurement reports under `docs/benchmarks/` for each machine, beside `gate-test-xdr.md`: **none**
- Note 4, run with `--dir` under `/tmp/lym-*` and `/tmp/zz-*` on both machines: C1 (documented usage; the runs themselves are the operator's)
- Earlier note, the fabricengine tier1 wall with the scan exclusions applied: none. It depends on the operator getting an exclusion granted ("if the operator can have that changed"), so no code change is asked for; no finding.
- Added-after-Discussion entries: none in the record.

**Findings**

1. MEDIUM, class `scope`. Location: the diff as a whole (no `docs/benchmarks/` file). The board entry's Note asks for a measurement report per machine under `docs/benchmarks/`, beside `gate-test-xdr.md`. The diff adds none, and the record has no Added-after-Discussion entry that drops or defers this.
   - Evidence: `_lym/discussion/decision-record.md:38` ("The result goes into a measurement report under `docs/benchmarks/`, for each machine, beside `gate-test-xdr.md`."); `_lym/landing/summary.md`, Design decisions, last bullet ("…are not part of this change: they need runs on the laptop and on the bare-metal machine, by the operator"); `git diff --stat` lists only `tools/machinecost/*`.
   - Fix: leave the code in place. A fixer cannot produce real runs from both machines. Defer to the run's parent, which can either record the deferral with `lym loom decision add`, or have the operator supply the reports to land in this diff.

No unrequested scope: every change traces to a goal point or is forced by one.

Tool calls: none; the inherited context held the record, the summary, the diff and the diffstat.
