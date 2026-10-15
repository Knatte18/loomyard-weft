---
round: 3
focus:
  - "Reroute-cap halt at the Webster row: D3 gives Webster its first `on_reroute` edge, so shed's reroute-cap halt becomes reachable there, and its reason ends with the clause loomcli tells every row (`loomRerouteCapWayForward` in `internal/loomcli/wiring.go`). Settle whether the decision record states a way forward that fits a plan conflict for that halt, how it is supplied, and whether D5 lands the matching refusal-spec loom row and an acceptance criterion, as `PATTERN-refusal-way-forward` requires."
  - "Last-routed timing against the reroute cap: D3 has the Webster adapter write the last-routed findings when it returns `Stuck` with `Reroute` set, while shed may halt capped instead of taking the edge. Settle what happens when Webster is resumed after a capped reroute: whether the repeated-conflict halt can fire for a conflict no plan review saw, and whether D3's Bound and Open risks state the chosen behavior."
  - "Conflict-record lifecycle across a Webster call: D2 places the removal of the conflict record inside begin-batch. Settle whether every refusal path of begin-batch that precedes the removal point (invalid batch number, plan parse failure, state load failure, no run in progress, batch derivation failure) is covered by the record's stated lifecycle or by D3's stated exposure, and whether D2's claim about when a record exists matches that."
  - "Markdown structure of D3: settle whether the bullet list in D3 is closed by a blank line, so that the following paragraph, Bound and Rationale render as separate blocks."
---

Entries restate the sites each question depends on; none refers to an earlier round.
