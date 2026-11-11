```yaml
lens: fix-consistency
outcome: reported
```

No findings that reach MEDIUM or above. Task: check the change set's agreement with its surroundings (the fix-consistency lens).

**LOW**
- **Changed text:** `internal/gatecli/testmap.go`, `mapPackages` now adds the unmeasured jobs to `r.queued` on an error or signal. The `queued` field comment (about line 218) and the `Queued` comment in `internal/testcover/exitrecord.go` (lines 20-22) each list only some of the sources of queued work.
- **Summary:** Both comments omit the new source. The `testmap.go` comment lists jobs past the cap and build-lock skips. The `exitrecord.go` comment lists the cap, build-lock skips, HEAD moves and a pass that could take no slot. Neither names jobs left unmeasured when a pass ends on an error or a terminating signal.
- **Evidence:** the `testmap.go` comment reads "queued counts the queued work the call leaves: jobs past the cap and packages whose build lock another call held." The `exitrecord.go` comment reads "It also counts those a HEAD move left unmeasured, and every queued package of a pass that could take no slot." A reader trusting either would not expect `Queued` > 0 after a signal, which the new signal test asserts.

**Checked and consistent**
- The `PlannedHead` comment now matches `close`, which writes the record only when `r.head` is set.
- `gatecli/doc.go` (lines 1153-1154), the `dirs.go` comment and the `close` doc agree.
- `testcover/doc.go:58` and the `--upkeep` flag text need no change.
- The Refusals test's final no-record assertion matches the removal of `writesRecord`.
- The research proposals under `docs/research/` are dated records of one run, so they stay as written.

Tool calls (1):
1. A `grep` for planned_head, PlannedHead, testmap-exit and "exit record" across the repo's `.go` and `.md` files. My context did not hold the sibling docs and comments that mention the record.
