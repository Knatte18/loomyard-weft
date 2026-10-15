# Round 3 fixer-report — discussion

## Changes

- F1 (MEDIUM): fixed as proposed.
  Premise confirmed in `internal/shuttleengine/attach.go`: `minAge` is twice `StartupTimeoutS`, and `leftoverThenAgeVerdict` returns `verdictError` for a younger record, which `attach` reports with its "younger than" error.
  D8's attach way forward now re-steps the row once the record is older than twice `startup_timeout_s`, and says why: `lyx reed remove` leaves the record reading running, so the probe applies its age rule.
  The attach-refusal acceptance bullet now asserts the refusal names that wait.
- F2 (MEDIUM): fixed with the first option the `fix` key offers.
  D8 replaces the spawn-failure claim with the unreachability reason (Claude the one registered engine declaring all three requirements, no role `Requires` beyond the `ForkSubagents` implication).
  New R5 names `engine-model-choice` and `codex-engine` as the tasks that land the loom-section rows in `contracts/specs/refusal-spec.md`, covering the `Start` refusal's alias way forward as well, which the review noted has the same gap.
- F3 (LOW): fixed as proposed; D4's attach bullet carries the finished-run exception, the reason left to D8.
- F4 (NIT): fixed as proposed; D7's Bound states every condition under which the audit and parse engine can differ, and R4 names only the owners.
- `support-log.md`: a Round 3 entry under Review rounds.

## Disputed

None.

## Deferred

None.
