# Support log — glyph-audit-tool

## Interview

Autonomous run: no operator answers; every pick is an auto-pick recorded in the question ledger.

### Turn 1 — exploration

Read the board entry, `kickstart-per-card`, `glyph-use-audit` and `report-timing`; both research directories' `README.md`, `COMMANDS.md` and the scripts; `tools/tokencount/tokencount.sh` and `tools/codestats/codestats.sh`; the trace line in `internal/websterengine/beginbatch.go`; the durable sink's location in `internal/logger/logger.go`; `pattern/PATTERN-fabric-git.md` and `pattern/PATTERN-shuttle-provider-seam.md`.
Findings that shaped the design:
- Every tool under `tools/` today is a Go program with a `.sh` launcher; this one is shell and jq by the board's word, so Go test files are the way the gate reaches it.
- The trace line lands in the pair worktree's `.lyx/logs`, removed with the worktree, so the prompt must stay the classifier for landed runs.
- The research scripts hard-code `/home/knatte/...` paths for the projects store, the code repo and the records repo.
- `PATTERN-fabric-git` binds only code inside the `lyx` binary and exempts read-only verbs; `PATTERN-shuttle-provider-seam` binds `shuttleengine` and `reedengine` only, and the tool reads Claude transcripts as `tools/tokencount` does.

### Turn 2 — question batches (auto-picked)

Batch 1: Q1 tool form, Q2 inputs, Q3 output and intermediates, Q4 arm classification.
Batch 2: Q5 pack check, Q6 plan section, Q7 tests, Q8 PATTERN departures, Q9 records default.
Recommendations and picks are in the question ledger; alternatives weighed are under Rejected alternatives.

### Turn 3 — lens review

All six forks returned.

- premises, MEDIUM, D3 trace `pack_layout` holds only `file`/`inline`/`none`, so "trace wins" relabels `raf` forks: fixed, trace is now a family-level cross-check (duplicates: board-fit, goal-scope, pattern-fit LOW).
- premises, LOW, D2/D7 defaults are not tokencount's resolution: fixed, D2 drops the tokencount claim and D7 names the default as a simplification of `fabricengine.RecordsWorktree`.
- premises, LOW, trace lines carry no worktree key: fixed, D3 takes the worktree from the logs directory holding the file.
- board-fit, MEDIUM, a new layout lands in `unknown` with nobody to add its marker: fixed, D3 makes an unknown trace value its own arm, and Constraints require a task adding a layout to add its marker row (duplicate: goal-scope MEDIUM).
- board-fit, LOW, `kickstart-per-card` spans arrive in a tool result: fixed in Open risks, with the comparison metrics named for that case.
- board-fit, LOW, note `glyph-use-audit` not named: rejected, it is design context, not a dependency or consumer the record acts on.
- goal-scope, MEDIUM, per-card metric parked as a risk: fixed, Open risks now states plainly that span reach is unavailable for that layout until it adds a span source and which metrics carry the comparison; designing a span source for an unbuilt tool-call format would guess at it.
- goal-scope, LOW, D4 reproduction claim untestable: fixed, reworded to classifier and span-hit rule carried over unchanged.
- goal-scope, LOW, `--arm-from` beyond the entry: fixed, the flag is removed since the trace no longer overrides.
- goal-scope, LOW, Go tests against "shell and jq only": fixed, a Scope line says the rule covers the tool.
- pattern-fit, BLOCKING, script-running tests in tier 1 break `tierpurity_test.go`: fixed, D8 puts them under `integration`.
- pattern-fit, MEDIUM, hermetic env belongs in `TestMain` with `tmuxkit.Main`: fixed in D8.
- pattern-fit, MEDIUM, tests would write into the worktree's `.scratch/`: fixed, `--work` flag in D1, tests pass `t.TempDir()`.
- pattern-fit, LOW, `-weft` departure reads as a PATTERN entry: fixed, named as `weftname`'s convention.
- pattern-fit, LOW, no design home for a test-only package: fixed, D1 names the script header comment.
- scenarios, MEDIUM, missing records repo with only a trace file: fixed, D7 says it only disables the cross-check; acceptance criterion added.
- scenarios, MEDIUM, several trace lines per batch: fixed, last one before the fork's start; criterion added.
- scenarios, MEDIUM, in-flight forks and truncated lines: fixed, D4 parses per line and counts in-flight forks as they stand.
- scenarios, MEDIUM, no-edit, no-span and `none` forks: fixed, D5 excludes them per metric with per-cell sample sizes.
- scenarios, MEDIUM, invalid input partly specified: fixed, D2 lists every exit-2 case and bounds run names.
- scenarios, MEDIUM, a `--runs` typo looks like an empty run: fixed, unmatched names listed in the header.
- scenarios, LOW, concurrent runs: fixed, D1 states one run per work directory.
- scenarios, LOW, partial report on failure: fixed, temporary name then move; criterion added.
- scenarios, LOW, degraded-path criteria: fixed, criteria added for D6's missing writer, unresolved batches and the missing cross-check.
- scenarios, LOW, mtime window shifts on resume: fixed, added to Open risks.
- leanness, MEDIUM, auto-mode assumptions repeat decisions: fixed, cut to the two not stated as decisions.
- leanness, LOW, notes repeat port sources: fixed, notes keep only scripts no decision names.
- leanness, LOW, D7 last paragraph repeats D3: fixed, only the trigger condition kept, moved into the batch-resolution bullet.
- leanness, LOW, "not bound" reasoning in Constraints: fixed, Constraints keep the actionable rules, reasoning moved to Turn 1.
- leanness, LOW, Scope out bullet for tokencount and timing: rejected, it marks the boundary with two neighbouring measurements a reader could expect here; shortened.
- leanness, LOW, interview turns duplicate the ledger: fixed, Turn 2 now points at the ledger.

## Rejected alternatives

- A Go tool like `tools/tokencount`: the board fixes shell and jq.
- Keeping the 2026-10-13 `base_on`/`mid` arms: they split one prompt shape by commit date; the purpose is today's layout against the next.
- Requiring `--records` and `--code` always: breaks the one-command goal; defaults from the hub geometry with overrides instead.
- The trace line overriding the prompt: its `pack_layout` cannot tell `raf` from `inline`, and trace files are gone for landed runs.
- A `--arm-from` switch: with the trace as a cross-check it has nothing to switch.
- Shipping the 2026-10-15 negative control as a mode: a one-time check of the checker.
- Skipping the tests when `jq` is missing: hides a broken tool in a green gate.
- Untagged script-running tests: tier 1 spawns nothing.
- A `.cmd` launcher: bash and GNU tools are assumed; Windows runs it under WSL.
- Writing intermediates to a system temp directory: escapes the project and is lost for drill-down.
- Designing a span source for `kickstart-per-card`'s card-start tool call now: its output format does not exist yet.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Tool directory and name — resolved, auto-pick: `tools/forkaudit/` with `forkaudit.sh` and jq filters, against one monolithic script; the research scripts are already split by stage.
- Q2 Input forms — resolved, auto-pick: `--since`/`--until` or `--runs`, exactly one; paths overridable; run names restricted to a safe character set.
- Q3 Output and intermediates — resolved, auto-pick: `.scratch/fork-audit.md` and `.scratch/fork-audit/`, overridable by `--out` and `--work`, matching the existing launchers.
- Q4 Arm classification — resolved, auto-pick: prompt marker table decides; trace line cross-checks at family level and names an arm only for an unknown layout.
- Q5 Pack correctness — resolved, auto-pick: port with the Go-only declaration rule, `unchecked` for other languages, no negative control.
- Q6 Plan glyph use — resolved, auto-pick: port per run in the report; the writer is the session with the most card writes.
- Q7 Tests — resolved, auto-pick: Go integration tests with the standard `TestMain`, temp `--out` and `--work`, skip on Windows, fail without `jq`, test-budget row.
- Q8 PATTERN fit — resolved, auto-pick: lyxdirs departure named in Constraints; fabric-git and provider seam not bound.
- Q9 Records repo default — resolved, auto-pick: repo root's `-weft` sibling; the premises fork confirmed pair worktrees share the hub stores, so `git log --all` sees every pair's records.
