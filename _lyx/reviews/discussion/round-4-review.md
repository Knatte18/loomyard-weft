---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:208
    summary: D17's dropped-notice fallback starts every ready board task at every wake, an unbounded routing edge that bypasses the orch's operator-approval rule for starting tasks
    fix: >-
      In decision-record.md D17, replace the bullet at line 208 with "- for a task the first check finds landed, the orch reads that task's `landed; newly ready:` line in the hub notice log D16 appends to, and starts each task it names that the board still shows ready and that has not started; any other ready task waits for the operator's approval, as the orch's board policy requires." Then add after line 208 a bound sentence, "Bound: the fallback starts only tasks a landing named as newly ready, so it never starts a task the operator has not released." In D16 line 191, change "the orch learns of the landing and of newly ready tasks at its next wake from D17's board checks" to "the orch learns of the landing from D17's board checks and of newly ready tasks from that notice-log line at its next wake". Add an acceptance row after line 254, "The orch stencil's claims test pins that the next-wake fallback starts only the tasks a landed task's notice-log line names, never another ready task."
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:190
    summary: D16 says the landed task's board entry turns done at teardown, but Finalize marks it done and the Land row prunes it before teardown, so D17's board-entry-is-done check never matches after Land
    fix: >-
      In decision-record.md D16 line 190, replace "the row is done, and the board entry turns `done` at teardown." with "the row is done; Finalize already marked the board entry `done`, and the Land row's prune removed it." In D17 line 207, change "whose board entry is `done`, or whose pair is gone" to "whose board entry is `done` or gone from the board, or whose pair is gone". Make the same change in the acceptance row at line 254.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:136
    summary: D11's exemption sentence, rationale and acceptance row state the audit's reach differently from its own body and the Scope bullet
    fix: >-
      In decision-record.md D11 line 136, replace "A Warn site in no loop needs no per-site verdict: such a site is out of this audit." with "A line at Info or above that sits in no loop or polled path needs no per-site verdict: such a site is out of this audit." In line 137, replace "the audit narrows that to the ones in a loop or polled path" with "the audit narrows that to the Warn calls in a loop or polled path and adds the Info and Error lines there". In the acceptance row at line 249, replace "No Warn or Info line" with "No Info-or-above line".
---

### [BLOCKING] D17's ready-task fallback starts any ready task, unbounded

**Location:** `_lyx/discussion/decision-record.md:208`

**Issue:** D17's second dropped-notice check reads "any task the board shows ready and not yet started is started", at every wake, with no bound stated.
The rubric's attack-surface question requires a Decision that introduces a routing edge to state what it lets through and what bounds it.
What this edge lets through is wider than a landing's ready set.
The board's ready test (`readySlugs` in `internal/battencli/land.go:145`) is a task with no status and no `depends_on`, so a task the operator just promoted with no dependencies is ready at once.
The orch's board policy (`contracts/stencils/orch/orch-template-role.md:149`) says when a task starts is the operator's call, and the orch starts one only after the operator approves that bundle.
Under D17 the orch would start such an unapproved task at its next wake, landing or not.
The check is not tied to the deploy-key-gated check, which is correct, but it is tied to no landing either.
A durable bound exists: `rowNotifier.send` (`internal/battenshed/rownotice.go:198`) appends the notice line to the hub notice log before the hold-back check, as D16 states, so the `newly ready` list of a dropped notice survives on disk.
D17's first bullet also says "the newly-ready check runs for it" without naming what that check reads; the fix gives it the notice-log line.

### [MEDIUM] The landed task's board entry is pruned at Land, not marked done at teardown

**Location:** `_lyx/discussion/decision-record.md:190`

**Issue:** D16 says "the board entry turns `done` at teardown".
Loom's Finalize marks the entry done through `MarkTaskDone` once the parent-side merge lands (`internal/landingshed/finalize.go:288`, wired in `internal/loomcli/landingdeps.go:133`).
Batten's Land row then runs `PruneBoard`, which calls `board.Prune(true)` (`internal/battencli/land.go:114`), and `Store.Prune` drops every done task (`internal/boardengine/store.go:743`).
Worktree-Teardown is the row after Land in `contracts/recipes/batten-recipe.yaml`.
So once Land completes, the landed task has no board entry, and D17's first check ("whose board entry is `done`") cannot match it.
Only "whose pair is gone" catches it, and that holds only after Worktree-Teardown removes the pair.
A task whose Land finished but whose teardown is still running, stuck or failed therefore reads as running after a dropped landed notice.
This is the premise the focus directive asked about: the durable baseline is the board entry plus the pair's existence, neither depends on the `deploy` key, and the premise as written names the wrong board state.

### [LOW] D11's audit reach reads differently in its exemption, rationale and acceptance row

**Location:** `_lyx/discussion/decision-record.md:136`

**Issue:** The Scope bullet (line 27), D11's first sentence and its plan-listing sentence agree: every Info-or-above line, Warn included, in a loop or polled path, each given a verdict.
The exemption sentence at line 136 differs in two ways: it names only Warn sites, and only "in no loop", dropping "polled path".
Read literally, a Warn site in a polled path that is no syntactic loop is exempt by line 136 and audited by line 135.
Line 137 says the audit "narrows" the board's Warn check, but the audit also adds Info lines, which the board did not ask about.
The acceptance row at line 249 says "No Warn or Info line", leaving out Error, which "Info-or-above" covers and which has a production call site (`logger.Error`).
An exempt site gets no verdict, as line 136 says, and that reading is consistent once the reach is stated the same way.

## Focus answers

- D16 and D17 on a hub with a non-empty `deploy` key (this hub's `landing.yaml` sets `./update-plugins.sh`): the orch can tell a landed task from a running one after a dropped notice once its pair is gone; the board baseline needs F2's fix to cover the window between Land and teardown.
  Both board checks read board state and pair existence, and neither depends on the `deploy` key.
  The deploy-recovery check reads `origin/main` against `origin/prod`, runs only while `deploy` is empty, and so never runs on this hub; that scoping is correct.
- D17's ready-task fallback runs at every wake and is not tied to the deploy-key-gated check, but it is tied to no landing at all (F1).
  A landing into a parent task branch is covered by the board-and-pair baseline: Finalize marks the entry done whatever the parent, and the Land row skips only the pull and the deploy when the parent is not the prime's branch (`internal/battenshed/land.go:193`), still pruning the board and reading the ready set.
  The deploy-recovery check rightly leaves such a landing out, since nothing is deployed for it.
- D11 and Scope: see F3.
