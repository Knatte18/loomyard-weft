---
verdict: BLOCKING
findings:
  - id: F24
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:97
    summary: D4 points RecordsWorktree at the code worktree, so the transition seam's mid-merge probe MergeStateActive now reads the code merge state; any code merge in progress, D5's own step 2 to 4 window or a parked code conflict, makes every transition skip its commit and push and the run act unpushed, and a merge begun after the probe makes the path-limited records commit fail as a partial commit during a merge
    fix: "In D4, add a bullet deciding what a records commit does while the shared worktree is mid-merge, stating that `fabricengine.MergeStateActive` (internal/fabricengine/mergestateactive.go) reads the one worktree in a one-repository hub. Pick one rule and state its bound, for example: the transition waits as owed until the merge is committed or aborted and never skips, or records are committed through a path that git accepts during a merge. Pin the git fact in the Constraints' tool-fact list: `git commit -- <paths>` during a merge fails with `fatal: cannot do a partial commit during a merge.` (observed with git in a scratch repository this round; also stated in `failureDisposition`'s doc in internal/statuscommit/statuscommit.go). In D5, state whether steps 1 to 4 hold D4's write lock and what a crash between step 2 and step 4 leaves and how it resumes. Add a D4 acceptance row: a transition made while the worktree holds an unfinished code merge neither skips its commit and push nor lets the run act past the transition until the commit lands."
  - id: F25
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:185
    summary: D7's protected failed push and D8's failed view push both hold no run, but the transition seam halts the run on every push failure IsPushOwed does not read as owed, so either every prime run halts at its first transition while the prime branch is protected and a failed view refresh halts the task run, or the runs act unpushed against PATTERN-run-fsm's a transition commits and pushes before the run acts, with no PATTERN change declared; the record picks neither
    fix: "In D7, state what the transition seam (`internal/statuscommit`) does when the prime's records push fails as `protected`: either the run halts at the transition with the protection named and its way forward, or the transition completes with the records unpushed. If it completes, add a `## PATTERN changes` bullet changing `PATTERN-run-fsm` to allow that one case and name `fabricengine.IsPushOwed` beside `fabricengine.FailedPushesOwedBy` as readers of the new classes. In D8, state that a failed view refresh is never returned into the pair push's error, so the transition's push result is the branch push alone. Rewrite the D7 and D8 acceptance rows that say holds no run to the chosen behaviour, each naming what the run does at its transition."
  - id: F26
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:158
    summary: The prime push guard admits a fabric merge only by its SHA recorded in the worktree's .lym, and the off-branch refusal reads the prime's branch recorded in .lym, but PATTERN-lym-files makes .lym a cache that reads as empty and refuses nothing and on which nothing a run depends; a lost .lym or a fresh machine turns fabric's own merges into operator commits, owes the prime push and stalls prime runs, and an absent branch record has no stated behaviour
    fix: "In D7's predicate list, replace the .lym-recorded SHA with a git-only read, for example: a merge commit fabric wrote is a merge commit carrying fabric's merge trailer whose tree equals its first parent's records paths over the clean merge of its two parents' records-stripped trees. Drop from D5 step 4 the recording of the merge SHA in the receiving worktree's `.lym`. In D7, keep the prime's recorded branch in tracked records under the prime's `_lym` or in a git config key of the prime written by `lym fabric clone`, and state what fabric does when that record is absent. Keep the D7 acceptance rows and add one: a fabric merge on the prime's branch made before the prime's `.lym` was removed is still admitted by the prime guard."
  - id: F27
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:232
    summary: D5 defines a commit's records paths as the union of its own and its parent's .lym-anchor, and D9 has readers outside fabric (verifytree, the gate green record, webster's walk) strip them through a leaf helper, so those readers must read the marker from git, which the record's own Constraint and PATTERN-cwd-resolution reserve to fabricengine; the record never says how the readers get the paths
    fix: "In D9, state that the leaf helper takes the records paths, or the anchors they derive from, as input and reads no git, and that a reader outside fabric gets a commit's records paths from one fabricengine read (named as a new general seam, marked (new)) or is handed them by its caller. Keep the `PATTERN-cwd-resolution` Constraint as written."
  - id: F28
    severity: BLOCKING
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:315
    summary: The Open risks bullet still leaves build now or spec for the rewrite as the parent's call, and the support log's Q6 still reads open, but two added entries settle it (Build the mode into the current fabricengine (F16), and The motif writes the design only), so the record hands the plan writer an open question its own Decisions answer
    fix: "In `_lym/discussion/decision-record.md`, drop the `## Open risks` bullet that opens Board note `clean-room-module-rewrite`, and `design-simplify-callers`. In `_lym/discussion/support-log.md`, change Q6 in `## Question ledger` to: Resolved by the parent: built into the current `internal/fabricengine` as a design a later task builds (Added after Discussion entries of 2026-11-18)."
  - id: F29
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    carries: F8
    location: _lym/discussion/decision-record.md:241
    summary: D9's walk bullet admits records-only commits but not D5's own strip merge-ins; webster's parentMergeRejection admits a merge only when its tree equals the clean MergeTree of its parents, which a D5 merge-in never does because step 3 restores the receiver's records and per-branch records conflict, so a parent merge-in during a webster batch, accepted today, refuses the batch
    fix: "In D9's walk bullet, add that a merge commit fabric wrote by D5's strip passes such a walk when its tree, records paths stripped, equals the clean merge of its two parents' records-stripped trees, as the held-work checks strip before they merge. Extend the D9 acceptance row: a parent merge-in made during a webster batch, with conflicting records on both sides, does not refuse the batch."
prior:
  - id: F1
    status: resolved
  - id: F2
    status: resolved
  - id: F3
    status: resolved
  - id: F4
    status: resolved
    answer: The fixer's reading of lacking from origin's ref of the pushed branch is accepted. The .lym-recorded merge SHA the fix chose breaks PATTERN-lym-files, which is raised as F26.
  - id: F5
    status: resolved
  - id: F6
    status: resolved
  - id: F7
    status: resolved
  - id: F8
    status: open
    answer: The fix admits records-only commits through the walk, but not D5's own strip merge-ins, which the walk still refuses; carried as F29.
  - id: F9
    status: resolved
  - id: F10
    status: resolved
  - id: F11
    status: resolved
  - id: F12
    status: resolved
  - id: F13
    status: resolved
  - id: F14
    status: resolved
  - id: F15
    status: resolved
  - id: F16
    status: resolved
    answer: "Settled by the added entry titled: Build the mode into the current fabricengine (F16)."
  - id: F17
    status: resolved
  - id: F18
    status: resolved
    answer: Rewriting a Warp-SHA that names a replayed commit to the fork point keeps every trailer on code it was judged over, without halting landings; accepted.
  - id: F19
    status: resolved
  - id: F20
    status: resolved
    answer: A new owed class with a notice fits better than deferred, which sends none; accepted.
  - id: F21
    status: resolved
  - id: F22
    status: resolved
  - id: F23
    status: resolved
---

### [BLOCKING] A code merge in the shared worktree skips or breaks every transition

**Location:** `_lym/discussion/decision-record.md:97` (D4), with D5's steps at lines 113–120.

**Issue:** The transition seam probes `fabricengine.MergeStateActive`, which checks `MERGE_HEAD` and the conflicted files in `RecordsWorktree(l)` (`internal/fabricengine/mergestateactive.go:30-42`, wired at `internal/loomcli/wiring.go:218-220` and `internal/battencli/commitstatus.go:161`).
Its doc comment says it is records-only on purpose, because a two-sided check "would freeze every status commit while a live warp conflict-resolution session runs".
D4 makes `RecordsWorktree` the code worktree, so in a one-repository hub that is exactly what happens.
A parked code conflict, or D5's own window between step 2 and step 4, makes the seam skip both commit and push with only a warning (`internal/statuscommit/statuscommit.go:60-69,96-105`), and the run acts past a transition that was neither committed nor pushed.
A merge that starts after the probe makes the path-limited commit fail.
I reproduced git's refusal in a scratch repository: `git commit -- _lym/x` during a merge printed `fatal: cannot do a partial commit during a merge.` and exited 128.
Three lenses found this independently. The walkthrough lens judged it unchanged from today because it read `MidMerge`, which checks both sides, rather than the probe the seam calls.

### [BLOCKING] D7's protected class and D8's failed view push leave the run's fate unstated

**Location:** `_lym/discussion/decision-record.md:185` (D7), with line 222 (D8) and the D7 and D8 acceptance rows.

**Issue:** `statuscommit.New` returns any push error that `Owed` does not read as owed, and the run halts at the transition (`internal/statuscommit/statuscommit.go:73-78,124-125`).
`fabricengine.IsPushOwed` treats only transport, deferred and teardown_lock as owed (`internal/fabricengine/pushanchored.go:40-56`).
So "holds no run" means, in today's seam, that every prime run halts at its first transition while the branch is protected.
D7's bound ("the prime's records stay unpushed and no other machine sees them") reads as if runs go on.
If they do go on, a run acts with its transition unpushed, a departure from `PATTERN-run-fsm` that the record does not declare.
D8's failed view push has the same gap whenever the refresh rides a transition's pair push.

### [BLOCKING] The prime guard and the off-branch refusal depend on state in .lym

**Location:** `_lym/discussion/decision-record.md:158` (D7), with line 120 (D5 step 4) and line 190 (D7).

**Issue:** `PATTERN-lym-files`, a listed constraint the record restates: "`.lym` holds only caches", "An absent cache reads as empty and refuses nothing", "Nothing a run, the orch or a landing depends on lives only under `.lym`".
D7 admits a fabric merge into the prime push only by its SHA recorded in `.lym`.
If that cache is lost, or the hub is on a fresh machine, fabric's own merges read as operator commits.
The prime push is then owed as `operator_unpushed`, and prime runs wait for the operator: an absent cache that refuses.
The prime's recorded branch also lives only in `.lym`, and the record gives no behaviour for its absence.
Git can decide the predicate itself, from the merge's tree against its parents' stripped trees.

### [BLOCKING] Readers outside fabric need the marker read from git

**Location:** `_lym/discussion/decision-record.md:232` (D9), with line 106 (D5) and the Constraint at line 283.

**Issue:** D5's records paths are the union of the anchors named by a commit's own `.lym-anchor` and by its parent's.
D9 has `verifytree`, the gate green record and webster's walk strip those paths through a leaf helper.
For a commit's parent the only source of the marker is git, and the record's Constraint, restating `PATTERN-cwd-resolution`, gives that read to fabric alone.
The declared `PATTERN-cwd-resolution` change moves the anchor's location only.
The record does not say how the readers get the paths, so the natural build breaks a listed constraint.

### [BLOCKING] Open risks and the support log still hold the question the parent settled

**Location:** `_lym/discussion/decision-record.md:315`, with `_lym/discussion/support-log.md:110`.

**Issue:** The first Open risks bullet ends "Whether the parent wants that, or a spec for the rewrite only, is the parent's call", and Q6 reads "Open, for the parent".
Two Added-after-Discussion entries settle it: the mode is designed for the current `internal/fabricengine`, and this motif writes the design only.
The rubric names an Open risks line that an added entry settles as a consistency finding whose fix drops the line.
It is raised at BLOCKING because an incremental round admits no new finding below that, and the plan writer would otherwise read the settled question as open.

### [MEDIUM] D9's walk rule still refuses D5's own merge-ins (carries F8)

**Location:** `_lym/discussion/decision-record.md:241`.

**Issue:** `reconcileHead` steps over a merge commit only when `parentMergeRejection` finds its tree equal to the conflict-free `MergeTree` of its parents (`internal/websterengine/gitwrap.go:132-142,177-199`).
D5 step 3 restores the receiving side's records, so the merge-in's tree never equals git's merge.
Per-branch records such as `_lym/fabric/origin.json` also make that merge unclean.
A parent merge-in during a webster batch, the case the walk exists for, is then refused.
The F8 fix covered records-only commits only.

## Rejected

- Interactions F-F (LOW, D5 cannot tell a hand edit of `.lym-anchor` from fabric's anchor-change commit): a fair observation, but it is below BLOCKING, and this incremental round admits no new finding below BLOCKING.
- Walkthrough's verdict that D4's mid-merge case is unchanged from today: it read `MidMerge`, which checks both sides. The seam calls `MergeStateActive`, which is records-only by its own doc comment, so the one-repository layout does change the outcome (F24).
- Bound-breaker's D2 note that a marker push that lands, followed by a refused `lym-bolt` push, leaves the marker on origin: the lens itself judged it LOW and did not report it, and it is below the incremental round's bar.
