---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md (D1, D4, D5, D6) against peer run-robustness
    summary: the design changes internal/battencli/overview_test.go, which run-robustness (awaiting Merge-Bouncer) also changes, and shares the battencli, loomcli and cmd/lyx packages with that run; the parent decides the landing order
    fix: the run's parent decides whether operator-followups proceeds now and rebases onto run-robustness once it lands, or waits for that landing before Plan; no change to the decision record is needed for scope
---

## Scope judgment

The design covers every part of the board entry `operator-followups`, no more:

- Runs overview shows a paused run as paused with the time since the pause, read from `lyx-halt`'s record: D1.
- Hold-notice deferral ends when the advisor is held or dead: D2, which finds most of it already met through hook-based session state and builds the remaining up-line ordering plus pinning tests.
- A paste over tmux's command size limit: D3 loads the buffer from stdin, a variant of the entry's `load-buffer` idea, with a test above the limit in the acceptance criteria.
- Operator shortcuts: D4 and D5 follow the entry's shape (embedded committed file, lyx argvs only, one command per shortcut in its own help group, `Short` naming the expansion, sequence stopping at the first failure, trailing args to the last argv, a test refusing name clashes and unknown commands).
  The entry's open question on flags and the prime shortcut's open name are settled in D4.
  D4's fresh root per argv departs from the entry's wording "through the same root", records the departure, and keeps the behaviour the entry asks for: the real command's flags, envelopes and exit codes.
  That is a design detail for Discussion-Review, not a scope deviation.
- Loom status pane takes fewer rows: D6.

Out-of-scope lines respect the related entries: `shuttle-ready-blank-pane` keeps its readiness fix, `ide-prime-setup` is untouched, `orch-continue`'s `--continue` reaches `prime` only by a later commit, `creel` would sit on reed's transport D3 changes.
`loom-checklist-stale-render` (the status pane not cleared on redraw) is a separate bug in the same pane; D6 changes only the strand's size and neither duplicates nor contradicts it.
No decision contradicts or duplicates a board entry.

### [MEDIUM] Code-file and package overlap with run-robustness

**Location:** `internal/battencli/overview_test.go`; packages `internal/battencli`, `internal/loomcli`, `cmd/lyx`

**Issue:** `lyx loom peers` lists `run-robustness` (awaiting Merge-Bouncer) as changing `internal/battencli/overview_test.go`: it renames the strict-seed case in `TestOverviewSource_ReadsThePrimesRunsFromDisk` to `future-key` and widens every expected line's slug column.
D1 adds the pause marker to `internal/battencli/overview.go`, and the record's constraints require extending the existing overview render tests, which include that file.
The two runs also both change the `internal/battencli` package (this run `overview.go`; the peer `arm.go`, `cleanup.go`, `land.go`, `wire.go`), the `internal/loomcli` package (this run `bootstrap.go`; the peer about fifteen files, not `bootstrap.go`), and the `cmd/lyx` package (this run `main.go`, `clitree_test.go`, `sandbox_coverage_test.go` and a new test; the peer `daemonwakeups_test.go` and `tolerantrecords_test.go`).
`convergence-plan` (running, Webster) also changes `internal/loomcli` (`halt.go`, `landingdeps.go`) and `cmd/lyx` (`gitrepoboundary_test.go`).
No decision contradicts a peer's: D1 reads `pauserecord.Read` and `Record.Covering`, which `run-robustness` keeps with the same signatures while it moves only `Update` onto `internal/state`, and the record's constraint on the pause record's tolerant decode already names `run-robustness`.
No PATTERN entry is shared: this run edits `pattern/PATTERN-cli-cobra.md`, the peers edit other background files.
The overlap is textual, expected at a rebase onto `run-robustness` once it lands.

Shared bookkeeping files, named only: `cmd/lyx/testdata/test-budget.yaml` (both peers change it) and `docs/overview.md` (`run-robustness` changes one line; this run adds a line naming the shortcuts).
No parent commits are listed.
