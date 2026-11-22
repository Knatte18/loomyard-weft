---
round: 2
focus:
  - "pattern/PATTERN-config-layers.md and the PATTERN.md entry: does the text still say only `lym config reconcile` removes a base-layer or worktree key the template lacks, while `configengine.Unset` removes any named key on the operator's `lym config <module> --unset <key>`?"
  - "internal/gateslot/config.go `LoadRefuseRaw` and both engines' `Prepare`: does a launch still refuse an invalid `refuse_raw` when another gate key also fails the full load, and does a test reach that path in each engine?"
  - "internal/configreg/configreg.go and internal/gateslot/reinterpret.go: does a hub whose board marker already records the gate reinterpretation get the scope-less `harden-live-copies` removal, or a documented manual route?"
  - "internal/gateslot/reinterpret.go `RetireHardenLiveCopies`: is an entry removed exactly when it stops the gate config loading, whatever its tiers and scope decode to?"
---
