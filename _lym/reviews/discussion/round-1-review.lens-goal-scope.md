```yaml
lens: goal-scope
outcome: reported
```

Task: goal-scope lens. I checked the record against its board entry and looked for any decision notes for this task.

**Goal points of the entry, and whether the record meets them**

- **Part 1, Expected: the dobby re-execs between ticks and survives every deploy.** Met by Decisions 1, 2 and 4.
- **Part 1, Expected: a failed re-exec is logged with its error in `dobby.log` and posted to the orch.** Met by Decision 3, which covers both a failed exec and a refused re-exec start.
- **Part 1, Expected: a test replaces the binary under a running dobby and asserts it re-execs and keeps its tidy in flight.** Met by the first Acceptance criterion (tier1).
- **Part 1, suggested fix directions.** All are taken:
  - the graced pre-check or dropping it (Decision 1);
  - a longer wait for a re-exec (Decision 2);
  - logging and posting a refused re-exec (Decision 3);
  - "a trace file or a Warn on re-exec", met through a `dobby.log` line rather than a trace file, which Scope Out explains;
  - the binary-replacement test with a `Live` probe loop.
- **Part 1, "Not shown, to check on the fix" (SIGKILL or OOM kill, and which process held the lock).** Not checkable without kernel or journal access. The record keeps it as an Open risk and leaves the heartbeat silence as the signal. No finding.
- **Part 2: logger retention, or a fixed file with its own size bound rotated to one previous file.** Met by Decision 5, the fixed-file option.
- **Part 2: log only changes (start or re-exec, run started, halted, landed or failed, a failing pass, tidy spawn or end).** Met by Decisions 3 and 6.
- **Part 2: a ten-minute heartbeat naming the runs and their states.** Met by Decision 6.
- **Part 3, Idea 1: "a dobby counts and tidies only the landings of a root or beam its machine owns".** Partly met. Decision 8 counts by the machine that made the landing, not by the machine that owns the worktree now. Scope Out excludes a new owner tidying landings that the previous owner made and never tidied. See finding 1.
- **Part 3, Idea 2: a tidy marks its landings tidied for every machine, so an adopt does not tidy them again.** Met in effect by Decisions 7 and 8. A landing is counted only on its landing machine, so no second tidy happens.
- **Part 4: its own gate key `adopt`.** Met by Decision 9.
- **Part 4, operator decision: a line per tier to the orch, a notice at the end, progress in `lym beam status`, no window or strand.** Met by Decisions 10 and 11, and Scope Out excludes a window or strand.
- **Unlicensed work or work that belongs to another entry.** None found. The trace-file question is left to `test-logs-leak-into-bolt`, as Scope Out says. `spawn-audit-log` and `workspaces` (the parent project) are not touched by any Decision.

**Decision notes**

- The `lym board list` titles show no note titled `Decision for dobby-and-adopt-hardening:`, so there was none to read.
- The only parent decision is the operator decision for part 4, inside the entry body, and the record follows it.

**Findings**

1. **MEDIUM, `scope`** — `decision-record.md`, Decision 8 Bound and Scope Out ("Tidying on one machine the landings another machine made and never tidied").
   - **Summary:** the entry's Part 3 Idea 1 asks that a dobby counts and tidies the landings of a root or beam its machine *owns*. Decision 8 counts only the landings its machine *made*.
   - **Effect:** after `lym adopt` moves ownership, the new owner never tidies landings the old owner left untidied. They wait until the old machine owns the worktree again, which may never happen. The record makes this departure from the Idea in an Out line, without settling it.
   - **Fix:** either make the new owner pick up landings the old owner left untidied (for example a hand-off at adopt), or tie the exclusion to the entry's wording and state why the Idea's ownership wording yields to "tidied once".
   - **Evidence:** board entry body, Part 3 "Idea" first bullet (`lym board get --body`); `decision-record.md` Scope Out last bullet and Decision 8 "Bound:" line; `internal/dobbyengine/tidy.go:153` (a tidy runs only from this machine's `LandedSinceTidy`).

Tool calls:
- `lym board list`, filtered: to find any `Decision for dobby-and-adopt-hardening:` note and the touching entries. My context held only the entry body, not the list.
