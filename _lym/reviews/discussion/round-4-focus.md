---
round: 4
focus:
  - "In decision-record.md, D4's route for a darn run that reaches Darn-Write with no decision record: check each hop of the stated route against CheckGotoTarget and gotoAdmitted in internal/shedengine/goto.go (goto only moves a run back from the current row), and check the claim that Drift-Baseline and Drift-Await are Done without capture for a run with commits past the branch point against internal/driftengine/doc.go. Settle whether one route serves every such run, and whether the D4 acceptance criterion and the Open risks line state that same route."
  - "In decision-record.md, D4's missing-record refusal text for `decision add` on a darn run: settle whether the way forward it names is valid for the row the run is parked on (at or before Record-Write, and Drift-Baseline, Drift-Await or Darn-Write), and whether an acceptance row reaches each text."
  - "In decision-record.md, D4's way forward for a record that differs from its committed copy: check internal/loomcli/decision.go for every path that leaves an appended, uncommitted record (including a failed commit after the append), and settle whether the stated way forward keeps the appended answer."
  - "In decision-record.md, D13's placement of the load-time retired-key check: check internal/configengine/layers.go (Layers, ValidateLocal) and internal/configreg/runsettings.go (PrepareRunSettings) for where a retired key in the local layer is refused today and what the refusal names, and settle that the Decision names a place that exists under PATTERN-config-layers and a way forward the refusal carries."
  - "In decision-record.md, D5's Darn-Write validate-only guard against D4's rule that Record-Write writes only a missing file and D6's bar on the writer editing the record: settle what a guard failure on a present but damaged record does, and whether a way forward exists that no step of the run forbids."
---

Each entry restates its sites so the reviewer needs no earlier round.
