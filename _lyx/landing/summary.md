# Add `lyx report timing`: per-run phase time, waits and stops by settler

`lyx report timing [--since <date>] [--by day|week] [--format json|markdown]` prints, mechanically from the hub's run records, where run time went.
It replaces the LLM-and-awk computation the operator used to run by hand, and sits beside `lyx report convergence`.
It is prime-only, read-only, takes no lock and makes no commit.

## What the report holds

- Per run: seconds per phase (`setup`, `discussion`, `plan`, `webster`, `merge`, `publish-finalize`, `land`, `other`), active time, waits by kind (`parent_review`, `pr_gate`, `escalation`, `halt`), sleep, and the span.
- Medians per phase over landed runs whose archive tag falls in the window.
- Day or week buckets (UTC days, ISO weeks) with active run-time, waits, sleep, and arbiter and alignment attempt counts and time.
- Stops by settler per bucket: `judge`, `shed`, `tackler`, `arbiter`, `alignment`, the caller label (`orch`, `driver`, `operator` or another role), `unknown` and `open`.
- Findings for records that do not decode, unknown row names, out-of-order stamps and abandoned records branches.
  A figure a run's records lack is absent, never zero.

`--since` is a UTC `YYYY-MM-DD` date, default seven days back; a date after today gives an empty report.
Flag refusals follow `convergence`: a JSON error, exit 1, before any cwd resolution.
JSON carries integer seconds; markdown renders every duration through the status pane's formatter (`12m05s`, `2d03h`).

## Caller label on goto and resume

To tell the orch, the LLM driver and the operator apart, the goto verb and the step verb that resumes a halted run now record who called.
The CLI layer classifies the strand through `internal/agentname` and passes the label down as a plain string, so `shedverbs` and `shedengine` gain no import and read no environment.
A goto writes it as an optional `by` field on its history entry; a resumed step records a pending label on the status, and the run's next history entry carries it as `by` and clears it.
The Go driver and batten's automatic steps pass `shed`.
The label is for reports only; no decision reads it.
Both fields are tolerant, so records from older binaries decode and read as `unknown`.

## Design decisions to know

- One engine: `ComputeTiming` in `internal/reportengine` runs over the same `Sources` snapshot and run selection as `convergence`, so both reports agree on which runs exist.
  `convergence`'s output is unchanged.
- Runs in flight are included: the snapshot lists pair records branches with no archive tag, resolved through `fabricengine` naming.
  A records branch with no tag and no live pair is left out with a finding.
- A row's interval runs from the latest earlier stamp across the merged loom and batten histories, since `at` is a completion time.
  The span opens at batten's first row, so time before `Seed-Child` lands in `setup`.
  Active plus waits plus sleep equals the span, and phases plus sleep sum to the span.
- Sleep comes from the systemd journal, read once by `reportcli` and passed to the engine as intervals; a missing journal subtracts nothing and sets `sleep_note`.
  A machine powered off rather than suspended counts as run time.
- Settler attribution joins stops to the Go driver's settle records (loom run's and batten's files) by producer, state and history length; the full rules are in the `reportengine` docs.
- Attempt time (arbiter, alignment) runs from `spawned_at` to the next row.
  The alignment attempt's `spawned_at` writer is unreliable and is not repaired here; a late stamp inside the span silently shortens that time.
- Hub pauses are not tracked; paused time stays in whichever phase it fell.
- Every run before this change reads its orch, driver and operator settles as `unknown`.

## Package moves

- `FormatSeconds` and `FormatMinutes` move from `internal/statuspane` to the new stdlib-only leaf `internal/durfmt`; the status pane's text is unchanged.
- The journal sleep parser moves from `tools/tokencount` to the new leaf `internal/sleepjournal`; `tokencount`'s subtraction is unchanged.
  The board's `engine-usage-readers` entry also rewrites `tools/tokencount`, so whichever lands second rebases onto the moved parser.
- The settle-record type and a tolerant decoder move from `internal/sheddriver` to `internal/shedrun`; `sheddriver` keeps the file's append and read.
- `gitrepo` gains `BranchRefs`.
- `PATTERN-leaf-packages` and the sandbox suite doc are updated.

## To check by hand

- Run `lyx report timing` in the prime against real hub records, in both formats and with `--by week`, and compare a landed run's phase seconds plus sleep with its span.
  Not run against live records.
- Resuming from a paused run: the label recorded on the status when a step resumes a paused rather than halted run may disagree with the halted-run case.
- The `shed` label test in the Go driver's deps overlaps the existing driver deps test; check for duplication.
- Batten rows between `Run-Shed` and the loom rows have no explicit phase mapping; check how the `setup` and `land` split classifies them.
