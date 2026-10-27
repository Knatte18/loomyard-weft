```yaml
lens: goal-scope
outcome: reported
```

Task: goal-scope lens. I checked the decision record against board entry T2, `dobby-and-adopt-hardening`, and looked for its decision notes.

**Goal points of the entry and whether the record meets them**

- Part 1, Expected: a dobby re-execs between ticks and survives every deploy. **Met** by Decisions 1 and 2, and the re-exec Acceptance criteria.
- Part 1, Expected: a failed re-exec is logged with its error in `dobby.log` and posted to the orch, never a silent exit. **Partly met.**
  - Covered: a failed exec, a refused re-exec start and a wiring failure after a re-exec (Decision 3).
  - Not covered: a `placeDobby` refusal of a re-exec'd image, and a fatal error out of `Run` right after a re-exec (state read, lock error, `recordStart`). See F1.
- Part 1, Expected: a test replaces the binary under a running dobby and keeps the tidy in flight. **Met** by the first Acceptance criterion (tier1) and Decision 4.
- Part 1, fix direction: drop or grace the pre-check, give a re-exec a longer wait, log and post a refused re-exec, leave a line at the re-exec. **Met** by Decisions 1 to 3. The record writes a line to `dobby.log` instead of adding a trace file, and the support log gives the reason.
- Part 2: route the log through `logger`, or bound it at its fixed place with one previous file. **Met** by Decision 5, which takes the second option, and the support log records why.
- Part 2: log only changes (start or re-exec, a run started, halted, landed or failed, a failing pass, a tidy spawn or end). **Met** by Decision 6.
- Part 2: a heartbeat every ten minutes naming the runs and their states. **Met** by Decision 6 and its Acceptance criterion.
- Part 3: a dobby counts and tidies only the landings of a root or beam its machine owns. **Met** by Decisions 7 and 8, which count by the machine that landed the run.
- Part 3: a tidy on any machine marks its landings as tidied for every machine, so an adopt does not tidy them again. **Met for "not again", narrowed for "once".** See F2.
- Part 4: a gate key of its own, `adopt`. **Met** by Decision 9.
- Part 4: a line per tier to the orch, a notice at the end (green or failed), and progress in `lym beam status`. **Met** by Decisions 10 and 11.
- Operator decision for part 4: no window or strand for the adopt check. **Met**, listed under Out and kept by Decision 10.
- Unlicensed work: none found.
  - The worktree-gone exit in Decision 1 repairs a bound the record itself introduces under part 1.
  - The `machinename` refusal in Decision 7 is the wiring part 3 needs.
- Work that belongs to another entry: none found. `test-logs-leak-into-bolt` touches trace logs under `_board/.lym/logs`, not `dobby.log` or `check.json`.

**Decision notes**

`lym board list` holds no note titled `Decision for dobby-and-adopt-hardening:`, so there was none to read and none I could not reach. The one parent decision is the entry's own "Decision for part 4 (operator, 2026-10-27)" section, which I read in the entry body.

**Findings**

- **F1**
  - Severity: MEDIUM. Class: scope.
  - Location: `decision-record.md`, Decision 3 and Open risks.
  - Summary: the entry's Expected says a failed re-exec is logged in `dobby.log` and posted to the orch, never a silent exit. The record leaves two re-exec failures out of that.
    - A `placeDobby` refusal of the re-exec'd image leaves only an envelope in `dobby.out`, with no `dobby.log` line and no post.
    - A fatal error out of `Run` before the loop, right after a re-exec, gets a line but no post. Decision 3 posts only for a failed exec, a refused re-exec and a wiring failure after a re-exec.
    - The record's own Goal says "never ends silently".
  - Evidence:
    - The entry body's Expected section ("A failed re-exec is logged with its error in `dobby.log` and posted to the orch, never a silent exit").
    - `internal/beamcli/cli.go:374-376`: a `placeDobby` refusal returns `output.Err` only.
    - `internal/dobbyengine/loop.go:113-130`: `Run`'s only fatal returns are pre-loop (MkdirAll, the lock acquisition error, `recordStart`). The loop at `loop.go:217-240` returns only nil.
    - `decision-record.md` Decision 3: the post list.
  - Fix: Decision 3 posts the failed-re-exec notice for a fatal `Run` error when the start reads as a re-exec. It either derives the log path and the re-exec test for the `placeDobby` refusals that come after `lymcwd.ResolveWorktree` succeeded, or states why the entry's Expected is bounded there.

- **F2**
  - Severity: LOW. Class: scope.
  - Location: `decision-record.md`, Decision 8 Bound and Scope Out ("Tidying on one machine the landings another machine made and never tidied").
  - Summary: the entry's part 3 asks that each landing is tidied once across machines. Under Decision 8, landings that machine A made and never tidied stay untidied until A owns the worktree again, which may never happen, so they are tidied zero times. This narrows the entry, and the record never names it as a deviation from the entry or a choice for the operator.
  - Evidence:
    - The entry body, part 3 Idea ("A tidy on any machine marks its landings as tidied for every machine").
    - `decision-record.md` Decision 8 Bound, and the Out list.
  - Fix: Decision 8's rationale states that the zero-tidy case departs from the entry's "tidied once" and why that is acceptable, or the record keeps a bounded catch-up, such as the new owner counting a landing whose landing machine's tidy is not recorded.

**Tool calls**

1. `lym board list`, filtered: to find this task's `Decision for` notes and any neighbouring entries, which my context lacked.
