# Round 1 fixer report — discussion

All seven findings were checked against the code and fixed in `_lyx/discussion/decision-record.md`.
The support log is unchanged: Interview item 4 stays as the deliberation, and F6 moved its requirement into D2.

## Fixed

- **F1** (MEDIUM, design): confirmed, since `recurringKeys(runDir, round-1)` keeps only keys open in two or more ledgers and is empty at round 2.
  D1 now names its source: a new Go-rendered facts list holding every key open in at least one earlier ledger, with the recurring-key list kept for CIRCLING.
  D1's Bound says what it relies on: the judge mapping a finding to a key in that list.
  The third Open risk now names the actual guard and states that no Go check reads the mapping.
  A new Acceptance criterion pins the list at round 2.
- **F2** (MEDIUM, design): confirmed against `Bouncer.Call`'s clear-and-re-seed, which archives a CONVERGED or settled-accept run directory and restarts at round 1.
  D2 now keys each entry by segment name: a settle replaces the segment's entry, appends one when none exists, and removes it when the ledger has nothing to carry.
  A later generation's `Done` supersedes the earlier one, and a replay writes the same bytes.
  The write runs before the circling accept's settle write, so a failed write leaves the accept pending for a retry.
  A failed `CONVERGED` settle is cleared and re-seeded as today, and the new generation writes the entry again.
  The replay and byte-identical criteria are restated to match.
- **F3** (MEDIUM, design): confirmed, since the `CommitPlan` closure in `internal/loomcli/wiring.go` stages `planparser.PlanDirRel()` only.
  D2 now makes the write and its commit one Go seam on the Bouncer, wired for all three segments, which commits the decision record alone through fabric's anchored commit; no existing pathspec widens.
  The commit criterion names that route.
- **F4** (MEDIUM, decision): D2 gains a `Bound:` covering four points.
  The write touches only its segment's entry, and every other byte is unchanged.
  A missing `## Open risks` heading or a failed `Validate` leaves the record as it was and errors.
  Entries are informational, never decisions to implement.
  Growth is capped at one entry per review segment.
  The criteria cover byte preservation and the missing-heading refusal.
- **F5** (MEDIUM, consistency): the Goal now states the rule "from round 2 on" and says round 1 keeps today's rule.
- **F6** (LOW, decision): D2 now states what an entry denotes on each `Done` path: under `CONVERGED`, a finding the fixer addressed that no fresh reviewer has seen; under a circling `accept`, possibly unfixed.
  The entry also names which path closed the round.
- **F7** (LOW, consistency): Auto-mode assumptions 1, 3 and 4 are cut to the pick and a pointer to D1 or D2, and assumptions 2, 5 and 6 are kept.

Touch-ups that follow from these fixes: the Constraints entry for `PATTERN-sole-parsers` and the plan-writer note say "write" instead of "append".
`lyx loom validate-discussion` passes.

## Disputed

None.

## Deferred

None.
