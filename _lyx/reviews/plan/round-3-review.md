---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/17-orch-role-run-loop.md
    summary: The escalations rewrite fixes circling but leaves the lyx loom goto and lyx shed goto line unchecked; both leave the run paused and batten never resumes it, so the orch must run lyx loom start after them as well
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/21-skill-skip-tables.md
    summary: The startup table's maxElapsed is zero for the unknown row and unset for the timeout row, so one zero value must mean both "at most zero" and "no bound" and the writer must invent a sentinel
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/plan/21-skill-skip-tables.md
    summary: The watcher table leaves the second clock advance past the timeout, the tick loop and the per-row final assertion outside the stated row fields, then offers a fallback to two tests, so the writer must invent the row shape or choose the fold
  - id: F4
    severity: LOW
    class: design
    location: _lyx/plan/17-orch-role-run-loop.md
    summary: The After landing claim for reading the friction notes and the driver's report matches today's unchanged text, so it pins nothing that changed, and the card never names the prime directories the reading now points at
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/20-drop-getter-delegation-tests.md
    summary: The card justifies keeping TestRunner_ContextTokens_Delegates as the one Runner success-path test and says other session tests reach only refusals, but the clear, compact and idle tests in the same file are success paths
---

### [MEDIUM] Card 17 leaves the goto line of the escalations section unchecked

**Location:** `_lyx/plan/17-orch-role-run-loop.md`, the Escalations bullet list.
**Issue:** The card checks the section against the recipes and corrects the circling verbs, which record a decision and resume nothing (`internal/loomcli/circling.go`).
The role stencil's other resume-dependent line has the same defect.
`lyx loom goto` and `lyx shed goto` "leave it paused" (`internal/loomcli/cli.go`, `internal/shedcli/cli.go`), and `internal/battenshed/doc.go` says batten never resumes a halted child, paused included.
After a goto the orch must run `lyx loom start` in the task worktree, which the card states only for circling.
The card claims the check is exhaustive ("the one other section that misstates a step"), so the goto line is a missed call site, not a deliberate omission.
**Fix:** State in the card that after `lyx loom goto` or `lyx shed goto` the orch resumes with `lyx loom start` in the task worktree, add it to the Escalations rewrite, and add a must claim for it in the `orch-template-role.md` row.

### [MEDIUM] Card 21's startup table cannot express both elapsed bounds with one zero value

**Location:** `_lyx/plan/21-skill-skip-tables.md`, first bullet.
**Issue:** The deleted tests assert `elapsed >= SkillLoadTimeout` for the silent skill and `elapsed == 0` for the unknown skill.
The card gives the row `minElapsed` and `maxElapsed`, sets `maxElapsed` to zero for the unknown row, and sets `minElapsed` to the timeout for the silent row.
The silent row has no upper bound, so its `maxElapsed` is also zero.
A plain `elapsed > maxElapsed` check fails the silent row, so the writer must pick a sentinel the card does not name.
The two specs also differ: the silent row sets `SkillLoadTimeout` and the unknown row does not.
**Fix:** Name the representation, for example `maxElapsed` as a pointer or a `bounded` flag, or give each row a single `wantElapsed` predicate kind (at least the timeout, or exactly zero).
State that the row's `spec` carries `SkillLoadTimeout` only for the silent row.

### [MEDIUM] Card 21's watcher table leaves its row shape to the writer

**Location:** `_lyx/plan/21-skill-skip-tables.md`, second bullet.
**Issue:** The deleted timeout test advances the clock by 99s, ticks, asserts `ReloadStep` is 0, advances by 2s and ticks again.
The card gives `beforeTimeout` for the first advance and names no field for the second, so the writer must invent it, along with the timeout's 100s value.
The two deleted tests also end differently: the unknown test ends the turns `t2` and `resumed` and calls `assertReload`, and the timeout test asserts the calls after the clear are two long with `skill:ly:board` second.
The card states both assertions, but not as row data, and then says the body "stays one short loop".
A loop with two assertion shapes needs a per-row assertion field the card does not declare.
The closing clause, "if a row needs more than those fields, the pair stays as two tests", leaves the fold itself to the writer, which is a decision left open in the plan.
**Fix:** Declare the full row: the `unknown` set, the clock advances as a list, `endTurns`, and a final-check kind (`assertReload` or the two-call check) with its expected value.
State that the table body is a short loop with one branch on the final-check kind, and drop the fallback clause, or state which outcome the writer must produce.

### [LOW] Card 17's After landing claim pins nothing that changed

**Location:** `_lyx/plan/17-orch-role-run-loop.md`, After landing bullet and the claims paragraph.
**Issue:** The stencil's After landing section already reads "pull, read the friction notes and the driver's report", so a must claim for "reading the friction notes and the driver's report after landing" passes before and after the rewrite.
What the card changes is that the reading names the prime's own durable `_lyx` directories, but it neither names them nor claims them.
It also asserts the landing carries those files there, without a `Uses` entry that grounds it; `internal/shedrun`'s drive-reports directory and the friction directory are the sources.
**Fix:** Name the two directories the stencil step points at (the run's drive-reports directory under the prime's `.lyx`/`_lyx` shed run directory and the friction notes directory), and make the must claim match that wording; add `internal/shedrun` to `Uses` if the stencil names the drive-reports path.
Check the durable-versus-ephemeral placement of each against `PATTERN-durable-vs-ephemeral-state` before the stencil says the landing carries it.

### [LOW] Card 20's reason for keeping the ContextTokens test is false

**Location:** `_lyx/plan/20-drop-getter-delegation-tests.md`, second bullet.
**Issue:** The card says `TestRunner_ContextTokens_Delegates` is "the one test of the Runner's success path through the `SessionCycler` capability, which the other session tests reach only for refusals and guards".
`TestRunner_ClearSession_PlaysScriptedSequence`, `TestRunner_CompactSession_PlaysSequenceForFocus` and `TestRunner_SessionIdle_ReturnsScriptedClassification` in the same file are success paths through that capability.
`ContextTokens` is the same pass-through shape as the `CompactedSince` test the card deletes, so the stated reason does not distinguish them.
Keeping or dropping it is the card's call; the justification must be true.
**Fix:** Either state the real reason `ContextTokens` stays (it is the only test of that pass-through returning the engine's reading) or delete it with the other pass-through and say so.
Whichever is chosen, drop the "one test of the success path" claim.
