```yaml
lens: fix-consistency
outcome: reported
```

Checked the change set for stale twins and sibling disagreements; two findings, both LOW.

**LOW — `_lym/landing/summary.md`, `## What changes` > `Failure handling`**
- **Summary:** It still lists only two failure cases, and no longer matches the code.
- **Evidence:** The text reads "A refused record, or an agent with no valid draft by its timeout, leaves the recorded description in place. Dobby then posts one line…".
  - The fix added a spawn failure, which drops the landings and posts a line (`describe.go` `endFailedSpawn`).
  - It added a draft that cannot be read, which now ends at the deadline.
  - It added a record that committed but whose pull request update failed, which posts a line naming `lym beam pr update <slug>` as the way forward, not "the next landing tries again".
- **Why it matters:** `internal/dobbyengine/doc.go` was updated for these cases, but the change description (the finding F6 touched) was not. A reviewer reads the old claim.

**LOW — `internal/dobbyengine/doc.go:29` (`A beam dobby ends when its worktree is gone`) and the `Deps.PrimeTidyShuttle` wiring comment `beamcli/dobbydeps.go:302`**
- **Summary:** The exit bullet does not say that the worktree-gone exit now also stops the describer agent.
- **Evidence:** The `PrimeTidyShuttle` comment in `loop.go` was updated. The package doc bullet says nothing on the agents stopped, and no sibling text mentions the tidy stop either. This is a gap more than a contradiction.
- **Why it matters:** The describer stop (F4) is documented only on the `PrimeTidyShuttle` field.

Checked, with no disagreement found:
- Remaining mentions of `readKeptTidy` and `rememberKeptTidy`: none outside tests.
- `docs/overview.md:294` and the stencil: consistent with the code.
- `DescribeState.OwedLine`: documented and used consistently.
- `BeamDescribeDeps.Record` doc: matches the new signature in `describe.go` and `dobbydeps.go`.

Tool calls:
1. `grep` for "board-tidy agent" and stop/gone wording in docs, contracts and internal: to find stale twins; my context had no doc search.
2. `grep` for `readKeptTidy` and `rememberKeptTidy` repo-wide: to find uncovered callers of the renamed functions.
3. `grep` of `doc.go`, docs and beamcli for worktree-gone and `PrimeTidyShuttle`: to find twin descriptions of the exit.
4. `Read` of `doc.go` lines 26-36: to see the exit bullet in full; my context held only the lines around 155-190.
