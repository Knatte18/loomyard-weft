---
round: 2
exclude_lenses: []
focus:
  - "Fix-task lifecycle in `_lym/discussion/decision-record.md` (D4, D5, D6, D8): walk a fix task through a re-run of `lym adopt` while it stands released and unstarted, through dobby's start step with `max_runs` held moved runs listed (dobbyengine counts every listed run not owned elsewhere), and through `lym batten cancel` at a green end (cancel writes draft and keeps the pair unless `--remove`). Settle whether the record leaves any state where the fix task cannot start, or where a front entry stays open and blocks the worktree."
  - "D7's closed state set in `_lym/discussion/decision-record.md`: Land's `PruneBoard` removes every done board entry, and a machine-failure stop halts a fix task. Settle whether `landed`, a halted run and `running_here` each have a reading the record's rule can produce from this machine's records and the reconciled board."
  - "Front-task writes in `_lym/discussion/decision-record.md` (D6, D8) against the declared `PATTERN-front-task`: settle whether the slug rule keeps one fix slug per subject given `boardengine.MaxSlugLength`, whether every board payload write path (upsert, upsert-batch, merge) is bound, how the in-process write meets the board's run-agent task guard, and whether `on_land` survives a dobby restart across the fix task's Land."
---

Round 1 raised blocking design findings in the lifecycle, state-reading and write-path areas above.
Every lens ran in round 1 and the BLOCKING and MEDIUM findings came from three of them, so none is excluded.
