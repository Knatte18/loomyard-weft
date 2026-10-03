# Fixer report — Discussion round 1

## Fixed

- **F1 (BLOCKING), report proves reaching the report step, not filing.**
  D6 now makes the report a success marker: the agent writes it only after filing succeeded, and on a failed `lyx selfreport create` ends its turn without one, so shuttle classifies `asking`, Reflect returns `failed` and D4 alerts.
  D5 defines a finished reflection by that rule.
  The first Open risk now names the failed-filing case correctly, a new Open risk records the refiling it causes, and acceptance criteria cover the reportless turn and the stencil instruction.
  The support log gains the rejected alternative of Go parsing a status out of the report.
- **F2 (MEDIUM), covered-notes record and selective archiving.**
  D5 places the record as a non-`.md` file beside the report, states that every archive moves only the covered notes, the record and the report, and states when a leftover report or record is discarded.
  An acceptance criterion covers a note written while a reflection runs.
- **F3 (MEDIUM), content bound on public issues.**
  D6 gains a content rule for issue bodies and notes that the repo is public; D1's and D3's bounds point at it.
  The stencil acceptance criterion requires the rule.
- **F4 (LOW), D2 and D5 composition.**
  D5 states the probe runs under the friction lock, D2's skip and the row's wait are unchanged, a killed process leaves no lock, and a record with no report and no live agent is reflected again at the cost of duplicate issues.
  An acceptance criterion covers the orphan record.
- **F5 (LOW), D7 and D8 restating scope.**
  D7 and D8 are deleted; "batten's recipe and teardown are unchanged" is a new Constraint; D9 is renumbered D7.
  The #274 history and Merriam's friction sources moved into the support log's interview turns, and its D7/D8/D9 references now point at Scope Out, Constraints and D7.

`lyx loom validate-discussion` exits clean after the edits.

## Deferred

Nothing deferred.
