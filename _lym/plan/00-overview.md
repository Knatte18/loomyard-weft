---
format: 5
approved: true
language: go
---

# Plan: coverage map uses — reach note, failure triage and diff mutation check

Put the per-test coverage maps of `internal/testcover` to three uses over one new reader, `internal/testreach`:
a reach note for a diff, which the writer gets from its own gate and the code review reads;
triage lines in the gate's failure report and every fixer's report;
and a mutation check of the lines a diff changes, run by hand through `lym gate mutate` for the trial the board note asks for.
The plan builds in the decision record's order: the reader and the reach note, then failure triage, then the mutation planner and `lym gate mutate`.
D10's `mutation` report step is designed in the record and built only after the trial pays, so no card builds it.

## Card Index

1 — testreach-changed-funcs — open `internal/testreach` with the finder of a diff's changed functions (D2)
2 — testreach-reader — the cross-package reader of the maps (D1)
3 — pattern-coverage-map-reader — add `PATTERN-coverage-map-reader`, its background and its guard test
4 — gate-config-reach — `gate.yaml` accepts `reach` and the three new keys, with template defaults (D12)
5 — reach-step — the `reach` report step: worktree map refresh, `reach.json` and the note (D3, D4)
6 — map-run-writes-index — the stamped index of the shared maps, which the prime's `lym gate test --map` alone writes after its maps (D1)
7 — reach-review-note — the `reach.json` reader, its HEAD match and the code review rubric's `reach_report` marker (D5)
8 — rerun-mark-writer-only — writer-only gate results bypass the cold fixer, and its re-run is marked (D3)
9 — reach-turn — the reach turn of the loomshed verify gate and the weaver batch gate (D3)
10 — reach-verb — `lym gate reach` for one function and over a red parent branch's landings (D6, D8)
11 — failure-triage — triage lines in the failure report (D7)
12 — triage-in-fixer-reports — every fixer's report carries its failed tests' triage lines (D7)
13 — gate-mutate — the changed-line mutant planner in `internal/testcover`, `lym gate mutate` and the trial's board note (D9, D11)

## Shared Decisions

- `internal/testreach` runs no git: callers hand it file contents (`FileSet`), read through fabric for a commit and from disk for the worktree.
  gatecli's `changedFuncsAt` is the one place that builds those sets, for the reach step, triage, `lym gate reach --test` and `lym gate mutate`.
- Every new reader of one package's map goes through `testreach.ReadPackageMap`, so card 3's guard keeps its pinned single-package readers to the three that exist today.
- The diff's base is the merge base with the parent branch, as the redundancy step takes it.
  The prime has no parent branch, so the reach step there diffs the working tree from HEAD, as the failure report's touched files already do;
  a reach report written in the prime is marked so and never matches a review's HEAD.
  Triage lines take no base in the prime: as D7 has it for a call with no parent branch, a mapped failing test's line says no change set exists, with no D8 command.
  `lym gate mutate` in the prime takes its base from `--base`.
- Once a main imports a package, a card creates a production function of it only together with its first caller from a main.
  `cmd/lym`'s tier1 guard `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` runs at every batch gate,
  so a function whose only caller lands in a later card would fail it if a batch closed between the two.
  A package no main imports lies outside the guard's analysis (`PATTERN-test-only-code`),
  so cards 1 and 2 build `internal/testreach` before card 5's gatecli code first imports it.
- The reach report is gatecli's, as the redundancy report is; packages that cannot import gatecli (loomshed, weaverengine) get the report through a closure the CLI wiring builds,
  `gatecli.ReachPendingOf`, typed by `verifytree.ReachPending`.
- The reach turn is one decorator, `verifytree.WithReachTurn`, wrapped around an existing verify gate:
  `shedrecipe.resolveGateEntry` wraps a `verify` element when `Env.VerifyReach` is set, and `weaverengine.batchGateSpec` wraps the `batch` entry when `Geometry.ReachPending` is set.
  So `loomshed.NewVerifyGate` and `batchGateClosure` keep their signatures, and the decorator learns the entry's attempts where the entry is built.
  A report counts only for the gate key that wrote it, so only rows whose key lists `reach` get a turn.
- The cold-fixer bypass is a result flag, `shuttleengine.GateResult.WriterOnly`, rather than a match on the findings' text; the reach turn alone sets it.
  The post-fixer re-run is marked through `shuttleengine.RerunMark` on `GateEntry.Rerun`, which the wrap sets and the decorator reads.
- Triage lines open with `gateslot.TriageLinePrefix`; gateslot, which already parses the failure report for `FailedTests`, reads them back with `TriageLines` for the reports that re-render failures.
- The caps the record names (five tests per function, five functions per triage line, 200 commits, the note's line cap) are named constants, not config keys.

## PATTERN changes

- `PATTERN-coverage-map-reader` — adds: Reading the coverage maps across packages or finding a diff's changed functions: only through `internal/testreach` (new); only the prime's `--map` run writes its index; a report step's review note is used only when it matches HEAD's tree. (test)
- `PATTERN-test-speed` — changes: Writing or running a test: via `lym gate test` by key; one `tiergate.Require` tier, lowest that fits; it pins behavior no other test covers, which the redundancy and reach reports check; skips, clocks, `t.Parallel`, `//lym:seam` and `//lym:watch` per the background. (test)
