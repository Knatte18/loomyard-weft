```yaml
lens: interactions
outcome: reported
```

Interactions lens: I checked every pair of decisions that act on the same state, then checked the acceptance criteria and Constraints against the decisions they touch.

**Pairs checked**
- D4 path-limited records commit × D5 merge window and parked code conflicts (same worktree, `MERGE_HEAD`): broken (F-A).
- D4 one write lock × D5 steps 1–4: broken. D5 never says the merge holds the lock, so a transition commit can run inside the merge window (F-A).
- D5 records-path union × D9 leaf helper × Constraint `PATTERN-cwd-resolution`: broken (F-B).
- D5 merge-in commit × D9 webster walk (`reconcileHead`): broken (F-E).
- D5 merge-SHA record in `.lym` × D7 prime guard × Constraint `PATTERN-lym-files`: broken (F-D).
- D7 prime branch recorded in `.lym` × D2 clone × `PATTERN-lym-files`: broken (F-D).
- D7 `protected` class × D6 owed and act rule × `PATTERN-run-fsm`: broken (F-C).
- D8 failed view push "holds no run" × D6 pair push × transition seam: broken (F-C).
- D7 `operator_unpushed` × the transition seam's owed retry: held. It retries without limit until the push lands, as D7 states.
- D4 `Warp-SHA` = HEAD before the commit × D7 rewrite to the fork point × D6 `OriginCodeTip` ancestor read: held.
- D5 "bring level with own origin" × D8 post-GitHub-merge alignment × D9 `ReadParentTips` code-only: held.
- D5 strip × D8 view tree with records from the merge base: held. The view's three-way merge leaves the parent's records unchanged.
- D3 Add deleting the run-records root × D5 merge-in and landing: held. Step 3 keeps each receiver's own records.
- D3 excludes × D4 path-limited commit: held.
- D6 teardown deleting the view ref × D8 view under the shared teardown lock: held.
- D7 replay on code branches × D11 by-entry Bolt replay: held.
- D5 marker-change refusal × D5 anchor-change commit × D7 "predicates are git reads": broken (F-F).
- D9 `HoldsWork` strip × D8 landed judgment: held.
- Acceptance criteria D7 ("protected … holds no run") and D8 ("failed view push holds no run") × the transition seam: broken (F-C).
- Constraints `PATTERN-gitrepo-client-boundary` and `PATTERN-fabric-boundary` × D8 force-push: held.

**Findings**

**F-A: BLOCKING, design.** Location: `_lym/discussion/decision-record.md:97` (D4 `RecordsWorktree`) and `:119` (D5 step 4).
- Summary: in a one-repository hub every transition commit is skipped while the worktree is mid-merge, so the run acts with no commit and no push. That breaks `PATTERN-run-fsm`'s "a transition commits and pushes before the run acts".
- Why: D4 makes `RecordsWorktree` return the code worktree itself. The transition seam's `MergeActive` probe reads `MERGE_HEAD` and the conflicted files of `RecordsWorktree`. A true answer skips both Commit and Push with only a warning. Today a parked code merge leaves `MERGE_HEAD` in the warp only, so the weft probe stays false.
- When it hits: in D5's step 2–4 window, and whenever D5 hands code conflicts to "today's handling", which parks the merge across producers.
- Git also refuses a path-limited commit during a merge ("cannot do a partial commit during a merge").
- Neither D4's lock nor D5's bound covers either case.
- Evidence: `internal/fabricengine/mergestateactive.go:30-42`, `internal/loomcli/wiring.go:218-220`, `internal/statuscommit/statuscommit.go:60-65,102-105,150-152`.

**F-B: BLOCKING, design.** Location: `_lym/discussion/decision-record.md:106` (D5 records-path union) and `:231-232` (D9 helper), against `:277` (Constraint).
- Summary: D9's single leaf helper must read `.lym-anchor` from git. Its readers outside fabric (`verifytree`, `gatecli`, webster's walk) need the records paths of commits, and D5 defines those as the union of a commit's own marker and its parent's. That breaks `PATTERN-cwd-resolution`'s "only `fabricengine` writes the marker or reads it from git", which the record's own Constraint restates.
- The declared change to `PATTERN-cwd-resolution` covers only the anchor's location, not this rule.
- Evidence: `pattern/PATTERN-cwd-resolution.md:18`; `decision-record.md:106-108,231-232,277`.

**F-C: BLOCKING, design.** Location: `_lym/discussion/decision-record.md:185` (D7 `protected`) and `:222` (D8 failed view push); acceptance criteria at `:321` and `:322`.
- Summary: D7 says a `protected` failed push "holds no run", and D8 says a failed view push "holds no run". But the transition seam halts the run at the transition on any push failure it does not read as owed.
- So every prime run halts at its first transition while the branch is protected, or the run is meant to act unpushed. That second reading contradicts `PATTERN-run-fsm`, and no PATTERN change is declared for it.
- D7's bound "the prime's records stay unpushed" implies the second reading.
- The record picks neither reading.
- Evidence: `internal/statuscommit/statuscommit.go:73-78,124-125`; `internal/fabricengine/pushanchored.go:40-56`; `pattern/PATTERN-run-fsm.md` ("A transition commits and pushes before the run acts").

**F-D: BLOCKING, design.** Location: `_lym/discussion/decision-record.md:120` (D5 step 4) and `:158`, `:190` (D7), against `:276` (Constraint `PATTERN-lym-files`).
- Summary: D7's prime-push guard admits a fabric merge commit only through its SHA recorded in the worktree's `.lym`. Fabric's off-branch refusal also reads the prime's recorded branch from `.lym`.
- `PATTERN-lym-files` says `.lym` holds only caches: "an absent cache reads as empty and refuses nothing", and "nothing a run … depends on lives only under `.lym`".
- With a lost `.lym`, every fabric merge in the prime reads as an operator commit. The records push turns owed (`operator_unpushed`) and prime runs wait until the operator pushes.
- The off-branch guard's behaviour with no recorded branch is unstated.
- A git-only predicate would avoid the cache, for example recomputing the stripped merge of the two parents and comparing trees.
- Evidence: `pattern/PATTERN-lym-files.md:244-253`; `decision-record.md:120,158,176-182,190`.

**F-E: MEDIUM, design.** Location: `_lym/discussion/decision-record.md:241-242` (D9 walk bullet).
- Summary: webster's walk accepts a merge commit only when its tree equals the clean merge of its two parents. A D5 merge-in from the parent has the task's restored records, not the source's, and records conflicts make the in-memory merge unclean.
- So any parent merge-in during a webster batch makes webster refuse the batch.
- D9 admits only records-only commits through the walk, not D5's strip merges.
- Evidence: `internal/websterengine/gitwrap.go:172-199` (`parentMergeRejection`), `:132-139`.

**F-F: LOW, consistency.** Location: `_lym/discussion/decision-record.md:122` and `:130` (D5).
- Summary: D5 refuses a source marker change "other than through fabric's anchor-change commit" but gives no git read that tells the two apart. By D5's own path rule, a hand edit of `.lym-anchor`, with or without a `_lym` move, is a records-only commit like fabric's anchor change. The only remaining signal is the commit message, which D7's "a commit message alone never admits" rules out as a predicate.
- Evidence: `decision-record.md:106-108,122,130,157-161`.

**Tool calls**
1. `grep MergeActive:` wiring and the `MidMerge` definition: my context lacked what the transition seam's mid-merge probe reads.
2. Read of `midmerge.go` and the `loomcli` `MergeActive` wiring: to confirm which probe loom's transition seam calls.
3. Read of `MergeStateActive`: to confirm it probes `RecordsWorktree`, which D4 makes the code worktree.
