# Support log — gate-builds-testmap

## Interview

### Turn 1 — exploration (autonomous)

The run is autonomous, so no operator turns took place; every pick below is an auto-pick recorded in the question ledger.
Four research subagents read the code:

- `lym gate testmap` today already runs one process per test, side by side up to tier 0's parallel clamped by memory, with an alone-rerun of a test that failed beside others.
  Its weight is around that loop: selector-based incremental planning, an audit round, a self-check, per-package build locks with a doubling-backoff wait, `--max` and run-record ordering.
- `lym gate test` renders each tier's shell command and runs it as one tied child; it never runs `go test -c`.
  The memory guard samples a tree for `.test` processes, and a tier's `memory_mb` wraps the tier shell in a systemd scope.
- `tmuxkit.Main` gives each test process its own run directory, `TMPDIR` and tmux socket, and no test listens on a fixed port;
  the one known cross-process collision is the check for a `.git` in the shared scratch root.
- The verb is named in `README.md`, `docs/overview.md`, three package docs, `internal/gateslot/template.yaml`, two PATTERN backgrounds and `lym gate redundant`'s refusal.

### Turn 2 — design settled (autonomous)

Picks Q1 to Q8 in the ledger settled D1 to D9.
The exploration also found the evidence D3 and D5 rest on:
the study ran 259 `internal/loomcli` tests as processes in 5.2 s, eight at a time,
and `tmuxkit.Main` gives each process its own run directory, `TMPDIR` and tmux socket, with no test on a fixed port.

### Turn 3 — lens review (autonomous)

Every lens returned a report.

- bound-breaker + interactions: D6 checked a clean tree only at the start, so a mid-run edit reached later maps under HEAD's stamp — fixed: D6 checks the package's and linked packages' directories and HEAD before each build and each write.
- walkthrough: D6 checked Go files only — fixed: any modified or untracked file under those directories.
- existing-paths + walkthrough + bound-breaker: a detached `--map` call keyed like a plain call and could join it or drop the flag — fixed: D1 makes `--map` part of the detached words, with a criterion.
- existing-paths + interactions: a map run's record would mix with manual runs in `lym gate report` — fixed: D7 writes no run record.
- existing-paths: green subtraction or write could carry a map run — fixed: D7 says it subtracts no green tier.
- walkthrough: the `memory_mb` clamp did not bind on an unlimited cgroup — fixed: D4 caps the count at `memory_mb` over the per-test cap regardless.
- interactions + bound-breaker: a guard kill beside others had no rerun rule, and D7's exit disagreed with D5 — fixed: D5 reruns killed tests alone, D7 judges the outcome after that rerun.
- interactions: the one-ahead build was not handled on an early end — fixed: D2 cancels and waits for it.
- walkthrough + bound-breaker: nothing built the prebuilt `lym` for tier 1 — fixed: D3 builds it once per call, with a criterion.
- bound-breaker: the slot hold had no bound — fixed: D7 states the `slots: all` bound.
- walkthrough + interactions: a failed build dropped old entries, a silent change for consumers — fixed: D7 keeps today's rule, and the `PATTERN-test-speed` constraint says the coverage-map section is rewritten.
- interactions: the `go-json` criterion covered tier 0 only — fixed.
- bound-breaker: the build-cache fact was overbroad — fixed: narrowed to the instrumented packages.
- existing-paths: old `.build.lock` files — fixed: D8 says they are inert.
- existing-paths: the report step reads only the hub's maps — rejected: unchanged behaviour, and map uses are out of scope.
- leanness #1, #2, #3, #4, #8: evidence and narrative in D2 to D5 and Scope — fixed: cut from the record, evidence kept in this log.
- leanness #5: build-cache numbers — fixed with the bound-breaker finding.
- leanness #6: notes repeated `relevant.md` — fixed: the two repeating bullets cut.
- leanness #7 + board-scope: the `redundancy-sweep-existing` open risk was no risk — fixed: dropped.
- leanness #9: auto-mode package order repeated D2 — fixed: dropped.
- board-scope: D1 runs map mode as its own pass rather than inside the tier command — rejected: a fair reading of the entry, reasoned in Rejected alternatives.

## Rejected alternatives

- Map mode inside each tier's shell command, so the tier's `memory_mb` scope wraps the test processes:
  the tier command is a hub-configured `go test` line, and per-test processes need a Go-driven runner;
  re-executing `lym` as a hidden worker under the scope adds a verb back, which the task removes.
- Keep incremental planning and only move the verb under the gate: keeps the part the operator calls too complicated, for a saving the payoff report measured as small.
- Allow `--map --tier 0` alone: a one-tier call would rewrite half a map or need a merge rule across tiers.
- Run packages in parallel as well as tests: two levels of parallelism under one memory clamp, for a build the study puts at minutes.
- Refuse a dirty tree once at the start: a file edited mid-run would still reach later packages' builds under HEAD's stamp.
- Drop a failed build's old entries: a silent change for `lym gate redundant`, which reads those entries today.
- Record coverage during ordinary gate runs: counters are process-wide and tests run in parallel, so no run can name a test.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): flag or verb? A `--map` flag on `lym gate test`, per the entry's idea. → D1
- Q2 (auto-pick): which tiers per call? Tiers 0 and 1 together, refusing `--tier` and `--upto`, so a map is never half rewritten. → D1
- Q3 (auto-pick): incremental planning? None; every package in the call is rebuilt whole, since the study puts a full build at about 16 CPU-minutes. → D2
- Q4 (auto-pick): how do the memory guard and the per-run cap cover the processes? The guard samples lym's own tree; the tier's `memory_mb` bounds the headroom the per-test caps are clamped to. → D4
- Q5 (auto-pick): collisions between parallel processes? Keep the alone-rerun and list each test that passed only alone. → D5
- Q6 (auto-pick): dirty files? Check the package's and linked packages' directories and HEAD before each build and write; a dirty package is skipped and listed. → D6
- Q7 (auto-pick): does a map run count as a gate pass? No: it writes no run, green, verify or failure record, and exits non-zero on a failed test. → D7
- Q8 (auto-pick): map location and format? Unchanged; last write wins between two runs. → D8
- Q9 (resolved, fact): does a coverage build share the normal build cache? No; a `set`-mode build reuses its own instrumented packages (smart-build study, Go 1.26.8). → Constraints
