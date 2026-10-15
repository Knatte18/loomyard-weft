---
round: 1
focus:
  - "internal/loomengine/config.go and config_test.go: the parent's per-run review budgets for darn and loom rewrote config keys and caps; the task retired `discussion_producer`/`plan_producer`, added `discussion_lenses`/`plan_lenses` and changed `MigrateConfig`. Settle whether the merged key set, strictness checks and migration still agree, and whether the budget keys are read through the task's new axes."
  - "internal/loomrecipe/loomrecipe.go and internal/loomcli/wiring.go, darnwiring.go: the parent changed review-budget and Bouncer/arbiter wiring while the task removed `applyDiscussionProducer` and the `DiscussionSeats` and `SingleLLM` rows. Settle whether any parent-added call, row or test still names a removed symbol or assumes the old engine swap."
  - "internal/shedrecipe/registry.go, recipe.go, entries_switched.go: the parent added switched rows (window, VS Code seed, VS Code open) and the task changed the `MultiLLM` row (first seat drops `name`) and removed registry rows. Settle whether the registry, the conformance test and the testdata recipes agree after both edits."
  - "contracts/stencils (darn-template-write, landing-template-describe-darn, loom-template-discussion), internal/darnengine/spec.go, internal/landingshed/describe.go: the parent added mid-run scope correction and PR-Align text while the task moved Darn-Write and Describe to seat tables. Settle whether the stencil placeholders, spec fields and seat prompts still match on both sides."
  - "Changed pattern files (PATTERN-config-strictness, PATTERN-leaf-packages, PATTERN-no-denied-recovery, PATTERN-gate-self-check-parity, PATTERN-daemon-wakeups, PATTERN-fabric-git): settle whether the task's new config keys, seat code and migration follow each one."
  - "docs/overview.md, contracts/specs/refusal-spec.md and the `doc.go` files of the overlapping packages: settle whether the two sides' edits left a stale or contradictory statement, such as a removed producer, chair name or retired key still named."
  - "Test economy across internal/loomcli/wiring_*_test.go, internal/loomengine/config_test.go and internal/shedrecipe tests: settle whether a parent-added test now duplicates a task test, and whether the combined behavior of budgets with zero-advisor seat tables has coverage."
---

Focus follows the overlapping files and the parent commits that touch them; each entry is an interaction question, not a verdict.
