---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:13
    summary: prune strips pruned slugs from every survivor's depends_on, so Decision 2's read-only-but-status rule either refuses prune during any run whose task depends on a done entry or lets prune edit a locked entry, and no Decision says which
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:44
    summary: Decision 4 refuses an upsert-batch when any entry it touches is locked, which contradicts Decision 3's status-only pass, and no Decision fixes what a batch or merge entry is judged against, so a batch listing one slug twice (status null, then a body edit) can unlock and edit in one write
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:27
    summary: the lock predicate locks abandoned, a terminal status the store's own Prune contract and tests already name, and the Auto-mode assumption that nobody sets non-run statuses ignores it and the set-status help's own active example
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:64
    summary: a claim written before Topology.Add stays as running · Worktree-Create when pair creation then fails, which locks an entry no run holds, and neither Decision 8 nor Open risks names this or its disposition
  - id: F5
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:49
    summary: the refusal's unlock arm applies only when no run holds the entry any more, but nothing states how the reader checks that, so an agent can clear a live run's status and edit before loom's next transition re-locks it, and Open risks does not record this
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:65
    summary: Decision 8 skips the claim for a done or absent entry, which loom also skips, so such a run is never locked, yet the Goal says the lock starts when a batten run claims the task and Decision 8's Bound covers only a failed claim write
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:55
    summary: Decision 6's heading says the fold refuses before any GitHub call, but intake import fetches the issue from GitHub before the board write, and only the comment and close come after it
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:33
    summary: the Rationale lines of Decisions 2 and 8 argue against alternatives that were not chosen (a field allowlist, a CommitStatus-time claim), a trail support-log.md already records under Rejected alternatives
---

### [BLOCKING] Prune collides with the locked-field rule

**Location:** `_lyx/discussion/decision-record.md:13` (Scope's list of mutating paths), `:31-34` (Decision 2).

**Issue:** `Store.Prune` (`internal/boardengine/store.go`) removes every done entry and then rewrites each survivor's `depends_on` to drop the removed slugs.
A running task that depends on a landed task is the ordinary case, so `lyx board prune` during a run changes a locked entry's `depends_on`.
Decision 2 locks `depends_on` and states that the store refuses any write that changes a field other than `status`.
Scope's list of mutating paths leaves out `prune`, so the record never says which way this goes.
If the rule holds, prune refuses for the whole run whenever the running task depends on a done entry.
If prune is exempt, Decision 2's "read-only but for its status" is false.
The plan writer has to choose one, and either choice contradicts a sentence in the record.

**Fix:** Add `prune` to Scope's list and state its behavior in Decision 2.
The likely answer is that prune's removal of a done dependency passes, because the dependency is satisfied and the field change carries no scope.
Add a matching Acceptance criterion: prune on a board where a locked entry depends on a done entry removes the done entry and drops it from the locked entry's `depends_on`, or is refused, whichever the Decision picks.

### [MEDIUM] Batch and merge comparison base is unstated, and Decision 4 contradicts Decision 3

**Location:** `_lyx/discussion/decision-record.md:44` (Decision 4), `:36-40` (Decision 3), `:59-60` (Decision 7).

**Issue:** Decision 4 says `upsert-batch` and `merge` "refuse as a whole when any one entry they touch is locked".
Read literally, a batch whose only item is a status-only upsert on a locked entry is refused, which contradicts Decision 3's rule that a status-only upsert passes.
`UpsertTasksBatch` applies each item to a projected snapshot in turn and accepts the same slug twice.
The batch `[{"slug":"x","status":null},{"slug":"x","body":"new"}]` therefore unlocks `x` in item one and edits it in item two, all in one write.
Decision 3 says a mixed write is judged against the stored status, but it never says whether "stored" means the entry as loaded before the batch or the projected snapshot.
If it is the projected snapshot, Decision 3's own Bound ("no single write both unlocks and edits") fails.
Focus 1 verdict: `merge` is sound if each check compares against the entry as loaded before the merge.
Its upsert runs before its `set_status`, and Decision 7 refuses a changing upsert into a locked entry, so no merge path edits and unlocks in one write.
Writing that comparison base down closes `merge` too.

**Fix:** In Decision 3 or 4, state that every multi-entry write (batch, merge) judges each entry by its net change from the entry as loaded under the lock.
A write is refused when that net change touches a non-status field or removes a locked entry.
Replace "touch" with that definition.
Add an Acceptance criterion for the duplicate-slug batch, and one for a batch whose only item on the locked entry is status-only, which passes.

### [MEDIUM] The predicate locks `abandoned`, and the Auto-mode premise overlooks it

**Location:** `_lyx/discussion/decision-record.md:27` (Decision 1), `:91` (Auto-mode assumptions).

**Issue:** Decision 1 locks any status that is set, non-empty and not `done`.
The store already treats `abandoned` as a terminal status: `Prune`'s doc comment says "An abandoned entry survives", and `store_test.go`, `board_test.go` and `render_test.go` use it, the render test on a dropped note.
Under Decision 1 an abandoned entry is locked: it cannot be edited or removed until someone clears its status, and Prune keeps it.
The Auto-mode assumption "nobody sets non-run statuses today" holds for the live board, which shows only `done` and `running · …`.
It does not hold for the vocabulary the board itself advertises: `set-status --help` and `merge --help` give `"status":"active"` as their examples.
Focus 3 verdict: no flow in the code or skills writes a non-run status today, but the documented vocabulary would lock an entry by accident, and the assumption does not cover it.

**Fix:** Do one of these and state the rationale.
Narrow the predicate to the run's own format: a status carrying ` · `, written only by loom and the claim.
Or name `abandoned` beside `done` as a released status.
Or keep the broad predicate, correct the assumption so it names `abandoned` and the help examples as locking, and record the consequence in Open risks.

### [MEDIUM] A claim whose pair creation fails leaves a dangling lock

**Location:** `_lyx/discussion/decision-record.md:64-70` (Decision 8), `:99` (Open risks).

**Issue:** The claim is written inside Worktree-Create, before `Topology.Add`.
When `Add` or the reconcile after it fails (`createRefusal`, an incomplete pair), the run stops with no pair, and the entry still reads `running · Worktree-Create`.
That entry is locked, and its status claims a run is in progress when none exists.
Decision 8's Bound covers only a failed claim write.
Open risks covers only "a run abandoned without landing", which reads as a run that got going.
Focus 4 verdict: the window from claim to loom's first transition is bounded on the success path, but not on this failure path.

**Fix:** State the disposition in Decision 8: Worktree-Create clears or restores the prior status when the row fails after the claim, or the dangling lock is accepted.
If it is accepted, add it to Open risks with its manual unlock, and add an Acceptance criterion pinning the behavior that was chosen.

### [MEDIUM] Nothing bounds who may clear a running entry's status

**Location:** `_lyx/discussion/decision-record.md:49` (Decision 5), `:40` (Decision 3's Bound), `:99-100` (Open risks).

**Issue:** Decision 3 bounds the unlock only as "a separate, visible `set-status` write".
Decision 5's refusal hands the unlock to whoever hit the lock, on the condition "when no run holds the entry any more", but neither Decision says how that reader checks the condition.
The 2026-10-06 incident came from an agent (the orch) editing a running entry.
The same agent, refused, can run `set-status` null, make the edit, and loom's next transition writes the status back.
`lastBoardStatus` resends only on change, so a clear between transitions is restored at the next producer change.
The lock then looks intact, and the edit has slipped through.
Focus 2 verdict: the stated bound is the whole contract, and it does not stop a live run from being unlocked.

**Fix:** Have the refusal's unlock arm name the check the reader runs first (`lyx batten status <slug>`, or the pair's absence).
Add an Open risk that an unlock on a live run is unchecked and only visible in the board's history, or bound `set-status` null itself if the design wants a hard stop.

### [LOW] A done or absent entry is never locked, contrary to the Goal

**Location:** `_lyx/discussion/decision-record.md:7` (Goal), `:65` and `:70` (Decision 8).

**Issue:** Decision 8 skips the claim for a done or absent entry, and loom's `SetBoardStatus` skips them too.
A run on a done entry therefore leaves the entry unlocked for the whole run, not just a window.
An absent entry created mid-run stays open until loom's next transition.
The Goal says the lock starts "the moment a batten run claims the task", and Decision 8's Bound mentions only a failed claim write.
Open risks says the `git-robustness` change will make batten refuse a done slug, but that has not landed.

**Fix:** Extend Decision 8's Bound to say that a done or absent entry is not locked during the run, and why, or cite the `git-robustness` refusal as the bound and state what holds until it lands.

### [LOW] Decision 6's heading overstates the ordering

**Location:** `_lyx/discussion/decision-record.md:55`.

**Issue:** `intakeImport` (`internal/boardcli/intake.go`) calls `selfreportengine.GetIssue` before `ImportIssue`, so a GitHub read always comes before the refusal.
The body of Decision 6 and the Acceptance criterion ("no GitHub comment or close happens") are right.
The heading "before any GitHub call" is not, and a plan writer who takes it literally could move the lock check ahead of the fetch with a second board read outside the lock.
Focus 5 verdict: the requirement the plan writer can act on is in the body and the Acceptance criteria.
Every behavioral Decision has a matching criterion apart from the gaps raised in F1, F2 and F4.

**Fix:** Change the heading to "refuses before any GitHub write" or "before the comment and close".

### [NIT] Rejected-alternative trail inside Decisions

**Location:** `_lyx/discussion/decision-record.md:33` (Decision 2's Rationale), `:68` (Decision 8's Rationale).

**Issue:** Decision 2 argues that one rule is "simpler … than a field allowlist", and Decision 8 explains why shed's `CommitStatus` cannot carry the claim.
Both argue against an option that was not chosen, and support-log.md already records both under Rejected alternatives.
Applying the completeness-before-leanness test: Decision 2's requirement (every non-status field is locked, including `recipe`, which Seed-Child reads after the claim) is already in its Decision text.
Decision 8's requirement (the claim is written inside the row, before `Topology.Add`) is also already in its Decision text.

**Fix:** Cut the comparison clauses, and keep the positive reason in each Rationale: Seed-Child reads `recipe` after the claim, and the claim has to come before pair creation.
