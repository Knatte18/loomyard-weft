# Fixer report: discussion, round 1

All six findings are fixed; `lyx loom validate-discussion` passes on the edited pair.

## Changes

- **F1 (fixed as proposed).**
  Premise confirmed: `loomPreRun` in `internal/loomcli/arm.go` refuses `run` without a status file, and only `seedAndCommitBootstrap` (start, step) seeds one.
  D3 now states that after a driverless seed the first driving call is `lyx loom step`, and `run`'s help names that order.
  D4 names the driverless route followed by `lyx loom step` in the missing-seed remedy, and brings `run`'s status-file refusal into scope, naming `lyx loom step` beside `lyx loom start`, with a new loom row in `contracts/specs/refusal-spec.md`.
  Scope, Auto-mode assumptions, Notes for the plan writer and the acceptance criteria follow, including one that walks the route: driverless seed, then `step`, drives a producer with no LLM driver.
- **F2 (fixed as proposed).**
  Premise confirmed in `b616a707-…jsonl` of the `git-robustness` project: a `tool_use` line at 2026-10-08T20:15 is followed by its `tool_result` on 2026-10-10 with no line between.
  D1 gains a 30-minute stall cap that makes an over-long interval idle whole, and its Rationale and Bound state what the cap gives up (a foreground call blocking past it, such as a foreground sub-agent call) and what it keeps (a stall under the cap).
  The "no crash transcript was available" risk is replaced by the residual sub-cap risk, and criteria cover a stall ending at a `tool_result` and at an assistant line, plus a sub-cap gap that stays working.
- **F3 (fixed as proposed).**
  Confirmed that compaction summaries are user lines with string content, marked `isCompactSummary: true`.
  D1 decides compaction is working time and excludes that line from the turn-start definition; a criterion covers a `/compact` line followed by its summary.
- **F4 (fixed as proposed).**
  Premise confirmed: `seedAndCommitBootstrap` rewrites the seed with the recorded parent and `shedrun.WriteSeed` refuses a disagreeing seed.
  D3 gains a Bound naming what a wrong parent causes, the help names where the right value is read (`parent_branch` in fabric's origin record), and the stranded case is under Open risks; a criterion covers the seed-stage refusal.
- **F5 (fixed as proposed).**
  D2 leaves out a transcript with no timestamped user or assistant line when a bound exists; an assumption and a criterion record it.
- **F6 (fixed as proposed).**
  Deleted D5's `validate-plan` sentence and D4's origin-record clause; both stand in the support log's Rejected alternatives.

The support log's Interview, Rejected alternatives and Question ledger record the new picks; the Review rounds section is left to the gate.

## Disputed

None.

## Deferred

None.
