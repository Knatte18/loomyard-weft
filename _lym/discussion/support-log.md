# Support log — fabric single-repo mode

## Interview

Autonomous run: no operator answers. Each turn is a design point, its evidence and the pick.

### Turn 1 — Reading the board

The entry is a task with four open questions: where `_lym` commits go, what a landing does with `_lym` and `.lym`,
which parts of the weft need a counterpart, and whether code outside fabric commits `_lym`.
No `Decision for fabric-single-repo-mode:` note exists.
Touching entries: `clean-room-module-rewrite` (wants this mode in a fabricengine rewrite), `design-simplify-callers` and `drop-compat-migration-code` (fabric cleanup, no conflict).
`lym loom peers` lists no peer touching fabric.

### Turn 2 — Exploration

Four subagents mapped the hub model, git use outside fabric, landing and pair push, and hub setup and the board.

- The weft is its own repository, sibling worktrees `<slug>-weft`, the Bolt an orphan weft branch; `_lym`/`.lym` junctions excluded in warp's `.git/info/exclude`.
- Nothing outside fabric commits `_lym`; `webstercli`'s sync goes through `Fabric.Commit`. Weft reads outside fabric go through `RecordsWorktree`/`RecordsRepoRoot`.
- `websterengine`'s audit depends on `_lym` resolving outside the worktree.
- Landing merges warp only; the weft branch is archived by tag at teardown, never merged.
- `verifytree` and the gate green record key on `HEAD^{tree}` and `HEAD`, and on `DirtyPaths`: a records commit on the code branch would invalidate them.

### Turn 3 — Where records commits go (auto-pick)

With `_lym` not excluded, a second branch leaves `_lym` as untracked dirt in every code worktree:
`git add -A` stages it and `verifytree.DirtyPaths` reports it.
With it excluded, the operator's rule breaks, and lym never force-adds.
Picked: the code branch (D4).

### Turn 4 — Landing (auto-pick)

Records on the code branch would reach the parent through any merge. Picked: strip by path at every merge (D5), and a records-free view ref as a pull request's head (D8).

### Turn 5 — Counterparts (auto-pick)

Records branch: the code branch. Board: orphan `lym-bolt`. Per-task branch: none, plus a view ref for pull requests. Transport: the one origin. Prime: guarded push and records-only replay (D7).

### Turn 6 — Code identity (auto-pick)

Gate and verify caches key on tree and commit; picked a records-free code identity applied in both layouts (D9), so code outside fabric needs no layout check.

### Turn 7 — Lens review

Six forks reviewed the first draft; every fork returned a report. Findings, merged across lenses:

- existing-paths 1, bound-breaker 3: `HoldsWork` and the held-work checks never read true after a strip merge. Fixed: D9 covers held work inside fabric and out.
- existing-paths 2, bound-breaker 3, walkthrough 1, interactions 9: after a GitHub merge of the view, Finalize's local merge pushes a new commit; the head-SHA check misreads the view. Fixed: D8 fast-forwards the parent instead and compares the head with the published view tip.
- existing-paths 3: replay paths would rebase task code branches. Fixed: D7 replays only records-only commits on every branch.
- existing-paths 4, bound-breaker 1 and 2, walkthrough 2: uncommitted receiver records block or are lost at merge, and fabric's dirtiness guards refuse. Fixed: D5 commits receiver records first, strips index and worktree, guards judge code only.
- existing-paths 5, walkthrough 3, bound-breaker 11: shared index and locks. Fixed: D4 path-limited commit, one write lock, bounded `index.lock` retry; D6 one push lock.
- existing-paths 6, bound-breaker 10: junction repair and weft-side excludes would act on real directories and hide code paths. Fixed: D3.
- existing-paths 7: anchor-change stop unclear. Fixed: D5 says it holds.
- existing-paths 8: hub workspace sibling folders. Fixed: D3; the orphan-sweep half rejected as call-site enumeration Plan owns.
- interactions 1, bound-breaker 4, walkthrough 4: view refresh has no base. Fixed: D8 reads `parent_branch` from the origin record; failed view push holds no run.
- interactions 2: owed prime push holds prime runs. Fixed: D7 states the hold and its way forward.
- interactions 3, bound-breaker 7, walkthrough 5 and 6: D7's bound false for landing pushes; landing merges unrecognisable. Fixed: bound narrowed to records pushes; fabric's merges carry a named trailer.
- interactions 4 and 5, bound-breaker 9, walkthrough 8 and 9: Bolt-based predicate misreads during clone or with `_bolt` lost, and its package was unsettled. Fixed: D1 reads the tracked marker, predicate in `lymcwd`; D2 commits the marker first.
- interactions 7, walkthrough 7, bound-breaker 12: `Warp-SHA` meaning. Fixed: D4 names HEAD before the commit; D7 rewrites it on replay.
- interactions 8: view lock mode. Fixed: D8 shared.
- bound-breaker 5: GitHub deletes the view head. Fixed: D6 tolerates a missing view ref.
- bound-breaker 6: D9's bound missed the "HEAD moved means work" direction. Fixed: D9 covers it and names webster's card checks.
- bound-breaker 8: `--single-repo` cannot spot a two-repository hub's code repository. Fixed as a stated bound and a docs warning in D2; no detection signal exists.
- board-scope 1 (decision): build now or only for the rewrite. Kept under Open risks for the parent, now naming `design-simplify-callers` too.
- board-scope 2: Goal said only fabric knows the layout. Fixed: Goal names `lymcwd`.
- leanness 1: D4 rationale argued the rejected option. Fixed: moved to this log; the operator rule is a Constraint.
- leanness 2: duplicate D9 risk. Fixed: Open risks bullet dropped.
- leanness 3: Auto-mode assumptions repeated decisions. Fixed: pointers.
- leanness 4: restating Constraints. Fixed: trimmed.
- leanness 5: `ResolveWithin` bullet stated nothing. Fixed: D3 states its behaviour.
- leanness 6: housekeeping line. Fixed: cut.
- leanness 7: `mktree` hint. Fixed: cut.
- leanness 8: docs criterion tied to D12. Fixed: docs in D2, criterion on D2.
- leanness 9: PATTERN background edits were an optional note. Fixed: moved to Constraints.

## Rejected alternatives

- Records on a second branch of the same repository, through a second worktree: needs junctions, which the task removes.
- Records on a second branch through a separate index file: needs `_lym` in the exclude, which the operator ruled out, or leaves it as permanent untracked dirt.
- Carrying `_lym` into the parent at landing: conflicts on every per-branch file (`_lym/fabric/origin.json`, `_lym/discussion/`, config) and breaks the per-branch records rule.
- Refusing pull requests in a one-repository hub: breaks "same as today from outside" and parents whose rules require pull requests.
- A records-detach commit on the task branch before a GitHub merge: removes the live records from the worktree mid-Finalize.
- A config key or a new marker file for the layout: can drift from the hub's real structure.
- The layout read from the Bolt sharing the prime's git directory (first draft): misreads during clone and with `_bolt` lost.
- A local merge after GitHub merges the view: pushes a commit a pull-request-only parent refuses.
- Converting existing hubs: no one asked for it.

## Review rounds

### Round 1

The review raised five blocking and eighteen further findings; every premise held against the code.
Picks made in the fix:

- F3, protected prime branch: `--single-repo` refuses at its marker push; a later protected refusal records a `protected` failed push that holds no run.
- F4, D7's predicates: a merge commit fabric wrote is one whose SHA fabric recorded in the worktree's `.lym`; lacking means unreachable from every origin ref.
- F10, records write between merge steps 2 and 3: step 3 compares and refuses, since records writers take no merge lock.
- F12, the prime's records branch: fabric refuses records commits off the branch recorded at clone; reset and stash stay a documented bound.
- F15, tool facts: pinned in a Constraint by documentation, an existing `gitrepo` test row, or the acceptance scenario.
- F16 (build now or for the rewrite) went to the parent; Q6 stays open.

### Round 2

The review raised five blocking findings and one medium; every premise held against the code.
Picks made in the fix:

- F24, records commits during a merge: committed through a temporary index and a HEAD move, which git accepts mid-merge,
  rather than the transition waiting out every code merge, since a parked conflict is when a run's Stuck transition matters most;
  fabric's merge abort resets to the moved start; a fabric merge crashed before step 3 is aborted by the next records commit;
  the transition waits only for a cherry-pick, revert or rebase, or a foreign merge whose index holds records unlike HEAD's.
- F25, protected prime push: the transition completes with its records unpushed, declared as a `PATTERN-run-fsm` change,
  since halting would stop every prime run for as long as the branch stays protected.
- F26, cache dependence: the fabric-merge predicate is read from git alone; the prime's branch lives in a git config key.

### Round 3

The review raised two blocking findings; both premises held against the code.
Picks made in the fix:

- F30, records lost to the abort of a merge crashed before step 3: the records commit is made first,
  from HEAD plus the worktree's records paths whose index entry equals HEAD's, and the abort resets onto it;
  a records path the source changed keeps HEAD's version, and a write to it after the crash is a stated loss.
- F31, re-running a merge verb: it keeps today's already-in-progress refusal, since aborting would drop a parked merge's resolutions;
  `mergeresolve`'s abort-first and the landing's pre-merge abort of its own parked merge-in stay the only aborts before a new attempt.

## Question ledger

- Q1 Code branch or separate branch for `_lym` commits? Resolved, auto-pick: code branch (D4).
- Q2 What a landing does with `_lym` and `.lym`? Resolved, auto-pick: stripped at merge, pull requests on a view ref (D5, D8); `.lym` is never tracked.
- Q3 Which weft parts need a counterpart? Resolved, auto-pick: D4, D6, D7, D8, D11.
- Q4 Does code outside fabric commit `_lym`? Resolved from the code: no; it reads weft through fabric's accessors and assumes code identity by SHA (D9, D10).
- Q5 How the layout is recorded? Resolved, auto-pick: the tracked anchor marker at the worktree root (D1).
- Q6 Build now or only for the rewrite? Resolved by the parent: built into the current `internal/fabricengine` as a design a later task builds (Added after Discussion entries of 2026-11-18).
