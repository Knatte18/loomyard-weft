---
round: 1
focus:
  - "internal/battencli/wire.go, SeedChild deps block: the parent changed ReadRunSettings and PrepareRunSettings to take and return battenshed.RunSettings, and the task rewired the same file for the Command engine; settle whether every closure, test double and envkit.FullEnv stub in the merged tree uses the new signatures and the task's changed wiring still reaches the parent's prepareRunSettings."
  - "internal/battenshed/config.go and template.yaml (task) against the parent's profile-aware Seed-Child in deps.go and seamchild.go: settle whether the task's new battenshed config keys and the parent's run-profile seeding assume the same layer order and the same strict or degrading load."
  - "PATTERN-config-strictness (parent added the runprofile exception): settle whether any config load the task added or changed in battencli, battenshed or shedrecipe follows the one-of-Load-or-LoadOrTemplate rule as the parent now words it."
  - "internal/lock/lock.go and lock_test.go, both sides touched: settle whether the task's lock change and the parent's LoopSpec.LockGrace removal and widened test bounds still agree on the lock's contract and timing."
  - "internal/configreg runsettings.go and its tests: the parent added profile layering and the task touched the tests; settle whether the task's test edits still pin the merged behavior and whether the merge left two tests covering the same case."
  - "Task's Command-engine and fabric detached-push changes against the parent's reed private-tmux-socket probe, creel typed-line mail and creelengine renewal journaling: settle whether any caller the task rewired relies on behavior those parent commits changed."
  - "docs/overview.md, internal/battenshed/doc.go: both sides edited the doc text; settle whether the merged prose still describes the merged code, in particular the run-profile seeding and the Command engine replacing the switched rows."
---

Seed focus drawn from the overlapping files and packages named in the rubric and a skim of both ranges' diffs; no entry pre-states a verdict.
