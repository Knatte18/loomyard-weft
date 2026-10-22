---
round: 1
focus:
  - "`docs/overview.md`: the task's `internal/lenses/` table row and the parent's `Darn-Write` reroute line merged in different places; check that the merged module table and execution-stack prose stay consistent with each other and with the code on both sides."
  - "`internal/testkit/envkit` (parent changed): do the task's new tests in `internal/lenses`, `internal/configsync`, `internal/configengine` and `internal/configreg` use envkit helpers whose behavior the parent changed, and do they still isolate the environment as they assume?"
  - "`internal/loomrecipe`, `internal/shedrecipe` and `contracts/recipes` (parent changed): does any recipe or fan key the parent added or renamed resolve through `internal/lenses` and its language layer as the task's lens and fan library expects?"
  - "`PATTERN.md` (task changed) against the parent's new code in `internal/shedrun`, `internal/shedverbs`, `internal/shuttleengine` and `internal/shellguard`: does the parent's new code follow the invariant the task added, and does the task's invariant contradict anything the parent added?"
  - "`internal/configsync` and `internal/configengine` (task) against any parent code that reads or writes the same hub config files, such as `internal/battencli` or `internal/landingshed`: does the parent rely on a config contract the task changed?"
---

Overlap is limited to `docs/overview.md`, and no package overlaps, so the risk is in cross-package use of shared helpers and recipes rather than textual conflict.
