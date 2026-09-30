---
verdict: APPROVED
rationale: "All nine round-1 findings are ratified and fixed in the current pair: D2's gate table is ordered first-match-wins with no-PR-required, closed-PR and malformed rows ahead of record matches; approve moves to the gate row (D4); the budget boundary (count >= budget) agrees with Routing.Bounces and episodeStuckCount; D5 baselines on HEAD, states blocked-rework recovery and crash re-entry; D8 defines a kind-qualified identity; the hand-made-fix change is a D2 decision; no blocking gap remains."
---

# Discussion-Review round 1 — judge verdict

The review report carries no `## Focus departures` section, so there are no departures to ratify or reject.

## Findings ratified

Each finding was checked against the rubric's Do-not-flag list and the writer stencil's exclusions before ratifying.
None asks for call-site enumeration, signatures or `file:line` citations, and none asks for rejected alternatives in `decision-record.md`.

- **F1 (BLOCKING), D2 gate table.**
  Legitimate: `checkApproval` in `internal/landingshed/publish.go` checks PR state before the record today, and the old table had no no-PR-required row.
  Fixed: the table is numbered, first match wins, and rows 1–4 (no PR required, merged, no PR or closed, malformed record) precede the record matches.
  D4 fixes the remove-then-write order, so at most one record exists, and row 5 outranks row 6 by position.
- **F2 (MEDIUM), approve's row check.**
  Legitimate: `internal/loomcli/approve.go:58` refuses unless `CurrentProducer == loomshed.NamePublish`.
  Fixed in D4 and in the acceptance criteria.
- **F3 (MEDIUM), budget boundary.**
  Legitimate.
  Fixed: D3 states five rejection rounds per task and an exhausted run stays awaiting at the gate; D4 refuses at count >= budget.
  Checked against `Routing.Bounces` (`internal/shedengine/progress.go:104`), which counts the segment's main-line row, and `episodeStuckCount`, which resets only on that row's own `Done`: the gate never returns `Done` between rounds, so the statements agree.
- **F4 (MEDIUM), append-only baseline and blocked-rework recovery.**
  Legitimate.
  Fixed: D5 baselines on the plan committed at HEAD and states both recovery paths; D4 accepts a run blocked at the rework row without consulting the budget.
- **F5 (MEDIUM), crash re-entry.**
  Legitimate: a requirement the plan writer needed as a decision, not a risk.
  Fixed in D5, with the Open-risks entry removed.
- **F6 (MEDIUM), batten identity.**
  Legitimate: `approvalIdentity` in `internal/battenshed/innerrun.go` is `approved_at + " " + head_sha`.
  Fixed: D8 qualifies the identity by record kind, and the acceptance criterion covers approve-after-reject within one second.
- **F7 (LOW), hand-made fix.**
  Legitimate relocation from assumption to decision.
  Fixed in D2 and the Constraints docs bullet; ledger Q13 points at D2.
- **F8 (LOW), acceptance criteria.**
  Fixed: "origin URL unusable" is in the `reject` refusal criterion.
- **F9 (NIT), D6 rationale.**
  Legitimate relocation under the completeness-before-leanness test: the clause carried no constraint beyond D6's Decision.
  Fixed: dropped from D6 and listed in the support log's Rejected alternatives.

## Residual observations, not blocking

D5's rationale ends "without making Master a planner", a brief echo of the Master-plans rejected alternative.
It is one clause, carries no misleading instruction, and does not justify another round.
