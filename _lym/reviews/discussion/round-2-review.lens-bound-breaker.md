```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I tried to break every bound, guard and claim about existing code in the decision record.

**Claims attacked**
- D1, layout read from the marker; `lymcwd`'s import set unchanged: held.
- D2, marker push before `lym-bolt`; a refused clone removes the hub directory: held. A marker push that lands followed by a refused `lym-bolt` push leaves the marker and seed records on origin's prime branch. That is LOW and not reported.
- D2, `--single-repo` creates only: held.
- D3, Add drops the parent's run-records root: held.
- D4, the path-limited commit never sweeps staged code: held, pinned by `TestCommitPathAllowEmpty`.
- D4, records commits work beside agents' work in one shared worktree: **broken** (finding 1).
- D4, `index.lock` bounded retry: held.
- D5, step 3 compares then refuses: held for writes. The restart path after a crash between steps 2 and 4 is broken; see finding 1.
- D5, the dirtiness guards judge code only after a records commit: held.
- D6, one push carries both, so no owed code tip: held.
- D7, "a commit message alone never admits a code commit": held.
- D7, the merge-SHA admission and the recorded prime branch kept in `.lym`: **broken** (finding 3).
- D7, the `protected` class "holds no run": **broken** (finding 2).
- D7, `operator_unpushed` read as owed so the run waits: held (`failedpush.go:276-292`, `statuscommit.go:124-128`).
- D8, publish idempotence, `parent_branch` refusal, force-push only `lym-view/`: held.
- D8, "a failed view push leaves a failed-push record that holds no run": **broken** (finding 2).
- D9, records-only commits pass webster's walk: held. A D5 strip merge-in does not pass it: **broken** (finding 4).
- D9, `HoldsWork` with the strip done before the in-memory merge: held.
- D10, D11: held.

**Findings**

1. **BLOCKING, design.** `_lym/discussion/decision-record.md:89`
   - Summary: D4's path-limited records commit is a partial commit, and git refuses a partial commit while a merge is in progress. In one repository, any code merge in the worktree blocks the run's transitions, and so does a crash inside D5's own uncommitted window.
   - Counterexample: an agent's parent merge-in stops on a conflict and leaves `MERGE_HEAD`, or a crash between D5's steps 2 and 4 leaves it. `statuscommit`'s mid-merge probe then reports active. The transition skips both commit and push and the run goes on, which breaks PATTERN-run-fsm's "a transition commits and pushes before the run acts". Any `lym fabric commit` in that window fails instead.
   - In a two-repository hub a code merge never makes the records repository mid-merge. The record says nothing about this case or about D5's restart path.
   - Evidence: `internal/statuscommit/statuscommit.go:60-69`, `:97-105`, `:150-152`; `internal/statuscommit/statuscommit.go:28` (the probe targets the worktree that carries the status file, which is the code worktree in one repository).

2. **BLOCKING, design.** `_lym/discussion/decision-record.md:184`
   - Summary: D7 says a `protected` failed push "holds no run". The transition seam does not wait on a push it does not read as owed; it returns the error and halts the run.
   - Counterexample: with the prime's branch protected, every prime run halts at its first transition. If the record means instead that runs proceed with their records unpushed, that departs from PATTERN-run-fsm's "a transition commits and pushes before the run acts", and the record declares no PATTERN change for it.
   - D8's "a failed view push leaves a failed-push record that holds no run" (line 222) breaks the same way when the view refresh rides a transition's push.
   - Evidence: `internal/statuscommit/statuscommit.go:73-78`, `:124-125`; `internal/fabricengine/pushanchored.go:40-57` (only transport, deferred and teardown_lock count as owed); `pattern/PATTERN-run-fsm.md`, "A transition pushes before the run acts".

3. **BLOCKING, design.** `_lym/discussion/decision-record.md:158`, also lines 120 and 190
   - Summary: D7's prime push guard admits a fabric merge only by its SHA recorded in the worktree's `.lym`, and its off-branch refusal reads the prime branch recorded in `.lym`. PATTERN-lym-files makes `.lym` a cache that "reads as empty and refuses nothing", on which nothing a run or landing depends.
   - Counterexample: a fresh machine, a lost `.lym`, or a machine that did not write the merge. Every fabric merge on the prime's branch then reads as an operator commit. The prime records push is recorded `operator_unpushed` and owed, and prime runs wait until the operator runs `git push`. With the branch record gone, the off-branch refusal has nothing to compare against, and the record leaves that behaviour undefined.
   - Evidence: `pattern/PATTERN-lym-files.md` ("`.lym` holds only caches", "An absent cache reads as empty and refuses nothing", "Nothing a run, the orch or a landing depends on lives only under `.lym`").

4. **MEDIUM, design.** `_lym/discussion/decision-record.md:241`
   - Summary: D9 lets only records-only commits pass webster's `reconcileHead` walk. A D5 parent merge-in is a merge commit, and it is judged by `parentMergeRejection`, which requires its tree to equal the conflict-free merge of its parents.
   - Counterexample: per-branch records such as `_lym/fabric/origin.json` differ between task and parent. Step 3 restores the receiving side's records, so the commit's tree never equals `MergeTree(parents)`, and the records conflict makes that tree unclean anyway. A parent merge-in during a webster batch, accepted today, is refused in a one-repository hub.
   - The F8 fix covers the walk only partly. It sits on a changed hunk, but I judge it below BLOCKING.
   - Evidence: `internal/websterengine/gitwrap.go:132-142`, `:177-199`.

**Tool calls**
- grep for `MergeActive` and the fast-forward dirtiness and merge-record readers (`statuscommit.go`, `loomcli/wiring.go`, `pull.go`, `mergestate.go`): my context did not show which worktree the mid-merge probe targets, or how a merge record blocks verbs.
