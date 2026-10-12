---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: decision
    location: cmd/lyx/main.go:98
    summary: The in-flight peer operator-followups changes cmd/lyx/main.go, including the same Available modules line, and the internal/loomcli package, both of which this run also changes; the order and conflict resolution need the parent's call
    fix: The parent decides the landing order of convergence-plan and operator-followups. The run that lands second merges the other into its branch at Merge-In and keeps both edits, so the Available modules line lists report and is followed by the shortcuts sentence, and both the reportcli.Command() mount and the shortcutcli.Mount call stay. No change to this run's design is needed.
---

### [BLOCKING] Code overlap with operator-followups in cmd/lyx/main.go and internal/loomcli

**Location:** `cmd/lyx/main.go:40`, `cmd/lyx/main.go:98`, `cmd/lyx/main.go:142`, `internal/loomcli/halt.go`, `internal/loomcli/landingdeps.go`

**Issue:** `lyx loom peers` now lists `cmd/lyx/main.go` and the `internal/loomcli` and `cmd/lyx` packages among `operator-followups`'s changed paths, which it did not at attempt 1.
In `cmd/lyx/main.go`, this run adds the `reportcli` import and mount and inserts `report` into the root's `Available modules:` line.
`operator-followups` rewrites that same line, adding a shortcuts sentence after it, adds the `shortcutcli` import and mount, and rewrites the file and package comments.
Both edits to the `Available modules:` line conflict textually, whichever run lands second.
In `internal/loomcli`, this run changes `halt.go` (the halt note is written through `haltnote`) and `landingdeps.go` (the checkpoint subject comes from `loomengine.LandingCheckpointSubject`).
`operator-followups` changes `bootstrap.go`, `bootstrap_test.go` and `doc.go` (the status strand's share).
In `cmd/lyx`, this run changes `gitrepoboundary_test.go` and `seamsignature_test.go`, and the peer changes `clitree_test.go`, `sandbox_coverage_test.go` and `shortcuts_test.go`.
The `loomcli` and `cmd/lyx` changes touch different files and different behavior.
No decision in either record contradicts the other, and the peer's shortcut names do not include `report`.
Under the escalation rule, a code file and Go packages both runs change go to the parent.

## Scope and alignment

The rework commit `1a0e9e54c` resolves attempt 1's reject.
`shedrun.ParseSeedTolerant` is deleted, `readLoomRecords` in `internal/reportengine/runs.go` decodes the seed through `shedrun.ParseSeed`, and `TestParseSeed` no longer pins a second parser.
No test pins the dropped "ReadSeed still rejects it" criterion.
`internal/shedrun/seed.go` is now unchanged against `main`, and `internal/alignment/records.go` keeps its `encoding/json` import for `ParseAttemptDir`.
This follows the parent entry `Added after Discussion (parent, 2026-10-12): Seed parsing follows run-robustness D4`.
`main` at `e5b56fe91` is an ancestor of the PR head.

The rest of the diff matches the goal and D1 to D12, as attempt 1 found.
`fd539e313` also shortens two unrelated PATTERN overview lines, `PATTERN-pane-text-injection` and `PATTERN-test-economy`, to bring the overview back under its headroom line after the `PATTERN-sole-parsers` line grew.
Their meaning is unchanged and no peer changes `PATTERN.md`.

## Other peers and bookkeeping

`lyx loom peers` lists no landed parent commits.

- `intake-bugs-1012` lists no changed paths.
  Its D5 adds a `status-unreadable: <error>` stop cause in `internal/shedverbs` and does not change the halt-note format that this run moves into `internal/haltnote`.
  This run's failure class, the reason's leading clause before `:`, groups that cause as one class.
- `cleanup-1012` lists no changed paths.
  Its tokencount cutoff-skew guard stays in `tools/tokencount`, which this run does not change.

Shared bookkeeping: `cmd/lyx/testdata/test-budget.yaml` and the module-tree and module-list lines in `docs/overview.md`, which `operator-followups` also changes, and the new `internal/haltnote` line in `pattern/PATTERN-leaf-packages.md`.
