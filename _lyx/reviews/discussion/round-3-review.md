---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:136
    summary: >-
      D6 claims the amendment seed leaves Shed's count at 1 whatever the previous generation spent,
      but Shed's budget block compares the old history before the seed is recorded,
      so a seed over a spent episode is blocked as budget exhausted instead of starting the generation
    fix: >-
      In D6, after the sentence that says the seed consumes one unit, state that a Stuck carrying `generation_start`
      skips Shed's budget comparison, as a BudgetExempt Stuck does, while it is still recorded as counted,
      so it can never take `budget_capped` and a seed over a spent episode starts the generation instead of halting the run.
      Add an acceptance criterion that an interrupted settle at the perch's last round (count equal to the budget),
      followed by a changed subject, runs the amendment's seed and does not block.
      Add a plan-writer note naming the `!output.BudgetExempt && episodeStuckCount(...) >= effectiveMaxBounces(...)` case in `internal/shedengine/run.go` as the site.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:139
    summary: >-
      D6's code-segment base reads the two sides differently from the identity of D5,
      the settled side is the settle commit while the identity hashed the working tree,
      and the current side is tracked files only while the identity also counts untracked files
    fix: >-
      Rewrite the code-segment half of D6's base sentence so both sides read the subject the way D5's identity does.
      Either name the settled side as the working-tree diff the settle recorded (stored or re-derivable from the record)
      and the current side as the same working-tree diff including untracked files that are not ignored,
      or narrow D5's identity to the same tracked scope and drop the untracked clause there.
      Whichever is picked, state that an uncommitted change at the settle is part of the settled side,
      so an amendment's reviewers never see already-reviewed work as new.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:138
    summary: >-
      D6 sets a document segment's base to the commit the settled artifact was read at,
      but the writer never commits and the Bouncer's Commit seam commits at the settle,
      so that commit predates the artifact and the amendment's reviewers would see the whole artifact as changed
    fix: >-
      In D5 and D6, name the document base as the weft commit that holds the settled artifact,
      read after the Commit seam returns (a told seam reads HEAD, since the Commit seam returns only an error),
      and written into the settle record when it is marked settled.
      State the interrupted-settle case: no commit holds the artifact yet, so the base is HEAD at the identity read
      and an amendment after an interrupted settle reviews the whole uncommitted artifact under the allowance.
      Add an acceptance criterion that a Plan-Bouncer amendment after a PR-Gate reject is told a base that contains the settled plan.
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:107
    summary: >-
      D5 counts every untracked file that is not ignored as a code change with no exclusion,
      and this worktree holds an untracked, unignored `.scratch_task.json` that no Go code writes,
      so a harness file can make an unchanged subject read as changed
    fix: >-
      In D5's code-segment bullet, bound the untracked clause with an exclusion set:
      the run's own state, as `UncommittedPaths` in `internal/websterengine` already excludes it,
      plus any file the told reader is configured to skip.
      Add one criterion that an untracked file outside the exclusion set changes the identity and one inside it does not.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:109
    summary: >-
      D5's Merge-In test says every commit since the settle commit is a Merge-In merge,
      but a merge of the parent brings the parent's own non-merge commits into that range,
      and nothing says how a commit is recognised as a Merge-In merge
    fix: >-
      Reword the condition to the first-parent commits since the settle commit,
      and state the recognition rule, a merge commit whose second parent lies on the recorded parent branch,
      so the told reader of the plan writer needs no further choice.
  - id: F6
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:110
    summary: >-
      The patch-id rule lets a Merge-In pass with no re-review whenever the parent's changes touch no line the task changed,
      and D5 does not state what that lets through or what bounds it
    fix: >-
      Add one sentence to D5 after the patch-id sentence naming what the rule skips,
      a parent change that interacts with the task's change without sharing a line,
      and what bounds it, the Merge-Review segment that reviews the merge-in and the verify tiers that run at Publish.
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/support-log.md:135
    summary: >-
      Question ledger Q13 still says the code identity sits outside the _lyx durable tree, which round 2 replaced in D5,
      and the Review rounds section stops at Round 1 though round 2 changed D5, D6 and D7
    fix: >-
      Rewrite Q13's answer to the current D5 form, the working-tree diff against the merge base with the parent,
      identified by the full diff hash and the zero-context patch-id, with the weft side holding the run's records.
      Append a Round 2 entry under Review rounds that states the verdict and the fixes,
      the Merge-Review generation record, the two settle times, the seed counted once, the code-identity form and the allowance validation.
      Add a Round 3 entry once this round's findings are fixed.
---

### [MEDIUM] Seed over a spent episode is blocked

**Location:** `_lyx/discussion/decision-record.md:136`, with `internal/shedengine/run.go:367`
**Issue:** D6 says Shed's count and the `Bounces` seam read 1 once the seed is recorded, whatever the previous generation spent.
That holds after the record, but the budget block runs before it: `stepLocked` compares `episodeStuckCount(st.History, …)` over the history it read at step 1, never the one with the seed appended, and the seed Stuck is not budget-exempt.
A Bouncer's count at round r equals r (the doc of `NewBouncer` says the seed takes one unit and the episode never resets inside a generation), so a generation that settled at the perch's last round, or at the soft cap by auto-accept, leaves the count at the budget.
If that settle was interrupted (no Done ends the episode) and the writer then changes the subject, case 2 archives and seeds, and Shed blocks the seed Stuck with `ReasonBounceBudgetExhausted` instead of routing it to the round producer.
That is the board incident's failure shape again for the changed-subject branch, and it contradicts D6's own claim and the acceptance criterion that an amendment's count reads 1 once its seed is recorded.

### [MEDIUM] Code-segment base disagrees with the identity

**Location:** `_lyx/discussion/decision-record.md:139`
**Issue:** D5 takes the code identity from the warp working tree, uncommitted tracked changes and untracked unignored files included.
D6 then tells an amendment's reviewers the settled task change as settle merge base to settle commit, against the current merge base to the working tree's tracked files "as the identity reads it".
Neither side matches the identity.
A change that was uncommitted when the settle ran is part of the hashed subject, yet it is absent from the settled side, so reviewers see it as new.
A new untracked file makes the identity differ, yet the current side lists tracked files only, so reviewers never see the file that triggered the amendment.
No code segment has a Commit seam (`commit_seam` accepts only `plan` and `discussion` in `internal/shedrecipe/entries_bouncer.go`), so the settle commit is whatever HEAD happened to be.

### [MEDIUM] Document base predates the artifact

**Location:** `_lyx/discussion/decision-record.md:138`
**Issue:** The discussion and plan writers never commit; the stencil for discussion says committing the two files is the loop owner's job, and the Bouncer's `Commit` seam does it at the settle.
The record is written before that commit, and D5 stores the base "the identity was taken at".
D6's document base, "the commit the settled artifact was read at", is therefore a commit that does not contain the settled artifact, at least on the artifact's first settle.
A diff from that commit shows the whole artifact as changed, which defeats "reviews only the change since".
The artifact is weft content (`_lyx` links into the weft checkout), so the record also has to say which repo's commit it means.

### [LOW] Untracked files carry no exclusion

**Location:** `_lyx/discussion/decision-record.md:107`
**Issue:** `.scratch_task.json` sits at this worktree's root, shows as `??` in `git status`, matches no `.gitignore` rule (`git check-ignore` reports none) and is written by no Go file in the repo.
D5's "an untracked file that is not ignored counts as a change" would put it into both the full-diff hash and the patch-id.
Any rewrite of such a file between settle and re-entry turns an unchanged subject into an amendment, against the acceptance criterion that an unchanged subject returns Done with no spawn.
`UncommittedPaths` already excludes the run's own state for the same reason.

### [LOW] Merge-In test reads wrong

**Location:** `_lyx/discussion/decision-record.md:109`
**Issue:** "Every commit since the settle commit is a Merge-In merge of the parent branch" holds for no real merge: the range includes the parent's own commits, which are not merges.
The text also names no way to recognise a Merge-In merge.
A plan writer who reads it literally builds a test that never passes, so every Merge-In reads as an amendment.

### [LOW] Patch-id skip has no stated bound

**Location:** `_lyx/discussion/decision-record.md:110`
**Issue:** The patch-id equality skips the re-review of a Merge-In whose parent changes touch no line the task changed.
Line-disjoint is not semantically independent, so this lets an interacting parent change through unreviewed.
The writer's stencil and spec require a Decision that downgrades a guard to say what it lets through and what bounds it, and D5 states the first only by implication and not the second.

### [LOW] Support log is stale against round 2

**Location:** `_lyx/discussion/support-log.md:135`, `_lyx/discussion/support-log.md:104-119`
**Issue:** Q13 answers "outside the `_lyx` durable tree", the restriction round 2 removed from D5 because `_lyx` is the weft side and no warp diff holds it.
The Review rounds section holds a Round 1 entry only, while the round 2 fixer report records changes to D5, D6 and D7 that no entry in the log carries.
The support log is the review's only memory of why a choice was made, so a stale ledger answer misleads the next round.

## Focus verdicts

- Merge-Review case 2: the generation record marks the new generation, the kept settle record then names an older generation and is not live, and the second Call reaches round 1's judgment without a second archive. Not a defect.
- Settle time: the Publish failure record's write time is compared against the live settle record's settle time. It closes at a round-by-round re-settle, whose time is later than the failure record, and stays open after a case 1 re-settle, which keeps the stored time. Not a defect.
- Amendment budget: with the seed counted once, the Bouncer's count at round r is r, so the soft-cap rule runs at round A and the hard cap at round A+1, as for a normal generation with a budget of A. The one gap is a seed over a spent episode (F1).
- Code-segment identity: the weft side holds `_lyx` (`_lyx` and `.lyx` are symlinks into the weft checkout and sit in the warp's `.git/info/exclude`), so a parent-side Merge-In changes no warp diff line the task owns, and the patch-id form settles the Merge-In case. The identity's untracked clause, the base's two sides and the document base are open (F2, F3, F4, F5).
- Allowance of 0: D7 states its validation (`CheckLocal` minimum 0 and no null, `CheckResolved` below 0), says it is not a guard key (matching `internal/reviewbudget/doc.go`) and states that every amendment, a PR-Gate reject's rework included, settles with no review. Not a defect.
