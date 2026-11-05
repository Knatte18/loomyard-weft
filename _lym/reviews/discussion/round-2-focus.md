---
round: 2
exclude_lenses: []
focus:
  - "Config meaning changes: for every hub config value whose accepted set or meaning the decision record changes, such as the drift scope values and the retired skip and verify_max_runs keys, settle whether the record states what a hub base or run_settings holding the old value does after this lands, and whether a PATTERN-config-reinterpretation reinterpretation or a refusing category covers it."
  - "In-flight runs: for a run already past Drift-Await, Drift-Expect or Drift-Recheck when this lands, whose scope record holds the old fields, settle what each Decision does with it, and how code tells the run's position."
  - "Refusal way forward: for every seal-check failure and every Stuck the record defines, settle whether the named way forward reaches that exact fault, per PATTERN-refusal-way-forward."
  - "Command mode: settle what the judgement entry, the seal check, a passing check and amendments each mean when the compare is opaque and code sees no cells."
  - "Guards that back a stated bound: settle whether the judge's tree guard and the expect re-entry check hold against a session that commits, adds an untracked file, or died mid-write."
---

Round 2 targets the design-class areas open after round 1; no lens is excluded because every lens run in round 1 raised a MEDIUM or worse finding.
