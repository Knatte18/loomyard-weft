# Fixer report — Discussion-Review round 1

All nine findings fixed. `lyx loom validate-discussion` exits 0 on the edited pair.

## Changes

- **F1 (BLOCKING), D2:** the gate table is numbered and evaluated in order, first match wins.
  Added rows for no PR required (`Done`, no GitHub call) and no PR found; closed PR and malformed record now come before the record matches.
  One catch-all row covers origin-URL, client and GitHub failures.
  Stated that the gate never returns `Stuck` except on a matching rejection, and that Publish keeps its no-PR-required `Done`, merged `Done` and closed `Stuck`.
  D4 now fixes the verb order (remove the other record, then write), so a crash leaves at most one record.
- **F2 (MEDIUM), D4:** `approve` accepts the gate row instead of `Publish`, and its help names the gate; acceptance criterion added; the migration assumption says an unapproved in-flight run runs `loom start` first.
- **F3 (MEDIUM), D3/D4:** the budget is five rejection rounds per task; `reject` refuses at count >= budget; D3's rationale now says an exhausted run stays awaiting at the gate, with the engine's block only a backstop.
- **F4 (MEDIUM), D5/D4:** the append-only check baselines on the plan committed at HEAD.
  D5 states the blocked-rework recovery (`loom start` retries; `reject` first replaces the findings), and D4 lets `reject` act at a run blocked at the rework row without consulting the budget.
  Acceptance criteria added for both.
- **F5 (MEDIUM), D5:** the round directory records `pr_number` and `head_sha`; re-entry that finds a committed round for the pending `head_sha` finishes only the tail.
  The Open-risks entry is removed, and the support log's interview turn 9 no longer points at an open risk.
- **F6 (MEDIUM), D8:** identity defined as kind + timestamp + head SHA in the one acted-on marker; the batten acceptance criterion covers approve-after-reject within the same second.
- **F7 (LOW):** the hand-made-fix behavior change moved from Auto-mode assumptions into D2, named in the Constraints docs bullet; ledger Q13 points at D2.
- **F8 (LOW):** "origin URL unusable" added to the `reject` refusal criterion.
- **F9 (NIT):** the `--fresh` clause is dropped from D6's rationale and added to the support log's Rejected alternatives.

## Deferred

None.
