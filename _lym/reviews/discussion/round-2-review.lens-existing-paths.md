```yaml
lens: existing-paths
outcome: reported
```

Task: existing-paths lens over the round-2 decision record. I found four breaks, all BLOCKING and all on hunks this round changed.

Paths traced, per decision:
- D4 (records commit on the code branch, `RecordsWorktree` returns the worktree): `statuscommit.New`'s mid-merge skip through `fabricengine.MergeStateActive` **breaks**. It now probes the code worktree's `MERGE_HEAD`.
- D4: `CommitEmptyWithTrailers` / `CommitPathAllowEmpty` with other paths staged **survives**, pinned by the `UnchangedPathWithAnotherPathStaged` row.
- D4: `commitEmptySnapshot` → `gitrepo.CommitEmpty` **survives on paper**. D4 states the new behaviour, and the change itself is Plan's job.
- D5 (merge-in commit with the receiver's records restored): webster's `reconcileHead` → `parentMergeRejection` clean-tree test **breaks**.
- D5: `HoldsWork` **survives**; the note strips both sides before the in-memory merge.
- D5/D7 (fabric merge SHA kept in the worktree's `.lym`): a lost cache, a fresh machine or another machine's merge **breaks** the prime guard.
- D7 (prime branch recorded in `.lym`): a lost cache **breaks**; the record says nothing of what an absent record does.
- D7 (`protected` class): the statuscommit seam's handling of a push failure that is not owed **breaks**. It halts the run.
- D7 (`operator_unpushed` class): `FailedPushesOwedBy`, `IsPushOwed` and statuscommit's owed retry **survive**, but only if the class is added to both class lists. `IsPushOwed` (pushanchored.go:45-56) is not named, and its mismatch is filed under F-C.
- D8 (failed view push "holds no run"): the same statuscommit halt path **breaks**.
- D6 (`pushOwedCodeTip`, `resetUnnamedTip` in pushclassified.go:258, `ReturnToPreviousTransition`): **survives**; D6 turns them off.
- D9 (`ReadParentTips`, `FastForwardPair` relation): **survives** as decided.

Findings:

**F-A — BLOCKING, design, `_lym/discussion/decision-record.md:97`**
- Summary: in a one-repository worktree a code merge in progress, or D5's own merge between steps 2 and 4, silently skips run transitions.
  - D4 makes `fabricengine.RecordsWorktree` return the code worktree.
  - Every transition's `MergeActive` is `fabricengine.MergeStateActive(location)`, which checks `MERGE_HEAD` and conflicted files in `RecordsWorktree(l)`.
  - So any code merge in that worktree makes statuscommit skip both commit and push, logged only at warn, and the run acts with no transition committed or pushed.
  - This departs from PATTERN-run-fsm's "a transition commits and pushes before the run acts".
  - A merge started after the probe makes the path-limited commit fail with "cannot do a partial commit during a merge".
- Evidence:
  - `internal/fabricengine/mergestateactive.go:30-42`
  - `internal/loomcli/wiring.go:218-220`
  - `internal/statuscommit/statuscommit.go:60-69` and `:96-105`
  - the same wiring at `internal/battencli/commitstatus.go:161`
- Fix: D4 decides what a records commit does while the shared worktree is mid-merge. For example, the transition waits as owed and does not skip, or records are committed with `--only` after the merge commits. It also states the bound, with an acceptance row: a transition during a code merge in progress neither skips nor halts the run.

**F-B — BLOCKING, design, `_lym/discussion/decision-record.md:241`**
- Summary: D9's walk rule admits records-only commits but not D5's own merge-in commits, so a parent merge-in during a webster batch is refused.
  - The walk's `parentMergeRejection` admits a merge only when its tree equals the conflict-free `MergeTree` of its two parents.
  - D5 step 3 restores the receiver's records, so the merge commit's tree differs from git's merge of the parents.
  - When records conflict, as `_lym/fabric/origin.json` does on every per-branch file, `MergeTree` is unclean and the walk refuses outright.
  - Today, before this change, a parent merge-in mid-batch is the case the walk exists to tolerate.
- Evidence: `internal/websterengine/gitwrap.go:113-144` and `:177-199`
- Fix: D9's walk bullet states that a merge commit made by D5's strip passes when its code tree equals the clean merge of its parents' records-stripped trees. Add an acceptance row: a parent merge-in during a webster batch with conflicting records does not refuse the batch.

**F-C — BLOCKING, design, `_lym/discussion/decision-record.md:185`**
- Summary: D7 says a `protected` failed push "holds no run", and D8 says a failed view push "holds no run". But the transition seam halts the run on every push failure it does not read as owed.
  - statuscommit returns the push error unless `Owed` reads it as owed, and `IsPushOwed` reads only transport, deferred and teardown_lock as owed.
  - So with a protected prime branch every prime run halts at its first transition.
  - A failed view refresh joined into a pair push halts the task run.
  - If "holds no run" is instead meant as "the run acts unpushed", it departs from PATTERN-run-fsm's "a transition commits and pushes before the run acts" with no declared PATTERN change.
- Evidence:
  - `internal/statuscommit/statuscommit.go:73-78` and `:122-126`
  - `internal/fabricengine/pushanchored.go:40-56`
  - `internal/fabricengine/failedpush.go:272-292`
- Fix:
  - D7 states what the transition seam does on a `protected` failure: halt with a way forward, or proceed. If it proceeds, declare the PATTERN-run-fsm change and name `IsPushOwed` beside `FailedPushesOwedBy` as readers of the class.
  - D8 states that a view push failure is never returned into the pair push's error.
  - Add a row for each.

**F-D — BLOCKING, design, `_lym/discussion/decision-record.md:158`**
- Summary: the prime guard admits a fabric merge only by its SHA recorded in the worktree's `.lym`, and D7 (line 190) keeps the prime's branch in `.lym`. Under PATTERN-lym-files, `.lym` holds only caches: an absent one reads as empty, refuses nothing, and nothing a run or landing depends on lives only there.
  - A lost `.lym`, a fresh machine, or a merge another machine wrote makes fabric's own merge commits read as operator commits.
  - The prime records push is then owed as `operator_unpushed`, and prime runs wait.
  - The record says nothing of what an absent prime-branch record does.
- Evidence: PATTERN-lym-files background ("`.lym` holds only caches", "An absent cache reads as empty and refuses nothing", "Nothing a run, the orch or a landing depends on lives only under `.lym`"), read in the manifest.
- Fix: D5 and D7 identify a fabric merge from git alone. For example, a merge carrying the trailer whose tree equals its first parent's records plus the clean merge of the parents' stripped code trees. Keep the prime's branch in tracked records or a git ref, and state what an absent record does.

Tool calls:
1. `grep` for `MergeActive:` providers and `MergeInProgress`: the manifest lacked who supplies statuscommit's mid-merge probe.
2. `sed` of `internal/loomcli/wiring.go:210-230` and `MergeInProgress`: to see the transition's `MergeActive` and Commit wiring.
3. `grep` of `MergeStateActive`: to confirm it probes `RecordsWorktree(l)`, which D4 repoints at the code worktree.
