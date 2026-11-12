# Support log — testmap-checked-in

## Interview

Autonomous run: no operator answered; every pick below is an auto-pick, recorded in the question ledger.

### Turn 1 — exploration

Three subagents read the coverage-map code, the weft topology, and the maps on disk.
Findings the design rests on:

- Maps sit at `<bolt>/.lym/testcover/<pkg>/map.json`, 211 files, 56 MB; every reader and writer passes the bolt dir.
- A map holds its commit, file hashes, block keys and per-test block lists; the stamp has no machine name or path.
- The weft is a separate repository; a pair's weft branch forks from its parent's weft branch and drops only run records.
- Weft branches never merge; teardown archives the tip with a tag.
- `testcover.PlanUpkeep` marks every current map changed when its commit moves, so a pass rewrites every map at a new HEAD.
- Sizes measured in this session on the 2026-11-12 set: indented 56 MB, compact 41 MB; gzip of the indented set 14.7 MB, of the compact set 14.2 MB.
  So a pass that rewrites the whole set adds about 14 MB compressed to git, and compaction saves disk more than git.

### Turn 2 — design picks

- Place: the existing weft branch, not an extra branch (D1).
- Inheritance and landing: fork carries the maps; the parent's upkeep rebuilds affected entries (D2, D3).
- Commit and push: the verb, at its end, scoped to `_lym/testcover/` (D4).
- Machines: owner machine only; stamp unchanged (D6).
- Churn: skip unchanged maps (D7); compact JSON (D8).
- Task worktrees may run the verb (D9); default off (D10); one import (D11).

### Turn 3 — lens review

Six forks reviewed the record; every fork returned a report.
Decision ids below are those of the revised record.

- bound-breaker, BLOCKING, uncommitted maps fail preflight's clean check: fixed, new D5 exempts `_lym/testcover/` with its bound.
- bound-breaker and walkthrough, no recorded owner lets two machines commit: fixed, stated as D6's bound with `lym adopt` as the way forward.
- bound-breaker and existing-paths, other `_lym` records commits sweep the maps in: fixed, D4 names them and why it is harmless.
- bound-breaker, a pair forked before the change has no maps: fixed, stated in D2.
- existing-paths, the named-package rebuild's commit was unclear: fixed, D4 covers both modes.
- existing-paths, `Format.Commit` must move with `Commit`: fixed, D7 says so and Constraints cites the `MissUnmarked` check.
- existing-paths, the per-pair uncommitted snapshot size: rejected, transient while a call runs and bounded by the set's size.
- existing-paths, D10's default flip is silent: rejected, the operator asked for off and a default change carries no reinterpretation.
- interactions and walkthrough, imported maps stay indented: fixed, D11 writes them compact.
- interactions, D11's removal never re-arms after a killed import: fixed, removal runs on any prime call once maps are committed.
- interactions, D7/D4 hand-run pass outside a run boundary: fixed, D4 commits under fabric's records write lock, and D5 keeps a live run's preflight clean.
- interactions, empty-maps window while upkeep is off: fixed, D2 states it.
- interactions, D6 claim tied stamp to ownership: fixed, D6 states the owner-upgrade rebuild.
- interactions, D1 criterion contradicted D11's read: fixed, the criterion excepts the import.
- walkthrough, the pass wrote through the junction yet commits: fixed, D4 writes into the weft path fabric hands it; Constraints restated.
- walkthrough, a killed pass may never be committed while upkeep is off: fixed, D5 makes the leftover harmless and D4 names the other committers.
- board-scope, BLOCKING decision, extra branch dropped: kept in Open risks for the parent.
- board-scope, decision, raddle split: kept in Open risks for the parent.
- board-scope, D11 deleted T2's exit record: fixed, D11 keeps `testmap-exit.json`.
- board-scope, D7 changes what a map's commit means for T2: fixed, D7 states the meaning and `planned_head`; Open risks names T2.
- board-scope, the sweep's hand rebuild: fixed, D9 says it stays in the task's weft.
- leanness, the extra-branch reasoning in three places: fixed, cut from D1 and Auto-mode assumptions.
- leanness, D2's cost lines: fixed, moved to Open risks.
- leanness, the dobby's exit-record read hid in Notes: fixed, moved into D4.
- leanness, measurement figures in D6: fixed, moved to Turn 1.
- leanness, facts stated twice: fixed, kept in Constraints.
- leanness, owner-machine constraint duplicated PATTERN changes: fixed, points there.
- leanness, Scope In restated the decisions: fixed, one line.
- leanness, D6 bundled two decisions: fixed, split into D7 and D8.

## Rejected alternatives

- An extra branch per pair in the weft repository with its own junction: it served a merge the operator's direction drops; it costs a third worktree per pair and new fabric topology for add, remove, archive and push.
- An extra branch per code branch shared across pairs: same cost, and a branch shared by pairs would need the merge the direction drops.
- Keeping the maps in the bolt: the operator ruled it out, since a branch must build its own maps.
- Putting a machine name into the stamp: every machine would rebuild everything; the toolchain fields already catch what changes coverage.
- Keeping the maps per machine in `.lym`: the operator wants them in git.
- Moving every current map to HEAD and accepting the churn: about 14 MB compressed per landing.
- Splitting each map into one file per test: more files and no smaller git cost; skipping unchanged maps cuts the churn more simply.
- Dropping `measured_at` and `built_at`: they change only when a map's content changes, so they add no churn once D7 holds; the audit orders by `measured_at`.
- Committing after each package to keep the worktree clean: many commits per full rebuild; D5's exemption is narrower.
- Refusing a pass when no owner is recorded: a single-machine hub with no owner would never get maps.
- Leaving `test_map_upkeep` on by default: it started an unauthorised 44-minute build.
- Rebuilding instead of importing the existing maps: 44 minutes for maps built at `5e41ee2b`, which has no Go change over `79f0eb4a`.

## Review rounds

### Round 1

- F1, BLOCKING, a task's pass registered hub-wide: fixed, D9 registers it under the task's slug with a same-worktree bound.
- F2, killed import lost the rest: fixed, D11 imports per package and removes an old map only once its package's map is committed.
- F3, machine-sharing premise: fixed, D6 and the Constraints fact name the machine env layer and the absent commit, both degrading to whole-package runs.
- F4, the audit skipped unwritten maps: fixed, D7 moves HEAD-comparing readers to the planned HEAD.
- F5, decision: held by the parent.
- F6 to F13: fixed in D4, D7, D8, D10, D11, Constraints and Open risks.

### Round 2

- F1, BLOCKING, build locks beside the exit record: fixed, D4 and the `PATTERN-lym-files` constraint put them under `.lym/locks/` through `lock.TrackedFileLockPath`.
- F2, hand calls in a task never registered: fixed, D9 registers every call in a task under the slug, with the teardown's bound.
- F3, arbiter-settled Open-risks lines: fixed, both dropped and D1's rationale cites "D1 stands".
- F4, a task's code-branch push had no gate: fixed, D4 runs the run-scope origin reconcile first, with a criterion.
- F5, uncommitted maps stopped the weft pull: fixed, D5 commits them before the weft moves, with a criterion.
- F6 to F10: fixed in D1, D7 and D11.

### Round 3

- F1, D4's push verdict had no home, exceptions or hold behaviour: fixed, D4 moves the verdict on the report to one `fabricengine` function, gives the verb only the pushes-off and prime skips, and makes a hold print one line with no pause or notice.
- F2, D5 overclaimed that the replay never stops on maps: fixed, D5 states the replay path and the live-pass bound, with a criterion.
- F3, no criterion pinned what D4's push carries: fixed, two D4 criteria, conditioned on the push going.
- Self-check: the verb drops loom's pause and Finalize skips, which would let a hand call push past loom's stop; a re-run that changes no map pushes a held commit.

## Question ledger

- Q1 Place: extra branch or existing weft branch? Auto-pick: existing weft branch (D1); raised to the parent, and the arbiter confirmed it (2026-11-12).
- Q2 Per code branch or per pair? Moot under D1: a weft branch is per pair, and pairs map one-to-one to code branches.
- Q3 How do two tasks touching one package merge a map? Auto-pick: they never merge; the parent's pass rebuilds (D3).
- Q4 Format smaller or more stable? Auto-pick: skip unchanged maps (D7) and write compact JSON (D8); keep the fields.
- Q5 Stamp per machine or shared? Auto-pick: unchanged fields, shared through git, owner machine writes (D6).
- Q6 Who commits, and two machines? Auto-pick: `lym gate testmap` on the owner machine only (D4, D6).
- Q7 `test_map_upkeep` default? Auto-pick: off (D10).
- Q8 Existing 56 MB in the bolt? Auto-pick: one import into the prime's weft (D11).
- Q9 May a task run the pass? Auto-pick: yes, by hand (D9).
- Q10 Uncommitted maps against preflight's clean check? Auto-pick: exempt `_lym/testcover/` (D5).
