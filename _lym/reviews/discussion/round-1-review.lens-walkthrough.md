```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens. I ran each decision's mechanism end to end on one concrete case.

**Walkthrough list**

- D1, case: task worktree with `.lym-anchor` at its root, Bolt present and missing. Purpose met. Note that today `readRecordedAnchor` reads the sibling path, so D3's change of read site is needed.
- D2, case: `--single-repo` on a fresh origin whose default branch refuses the push, followed by a rerun. **Purpose missed** (F4).
- D3, case: `Add` of a task branched from a beam parent that carries run records. **Purpose missed** (F3).
- D4, case 1: an agent has staged `src/a.go` and fabric commits one status file. Met: `StageAndCommit` runs `commit -m -- files`.
- D4, case 2: a tags-only, empty records commit while an agent's code is staged. **Missed** (F5).
- D4, case 3: the operator switches branch in the prime. **Missed** (F6).
- D5, case: a squash local merge of a task into its parent, with conflicting records edits. The steps hold. The held-work follow-on is **missed** (F7).
- D6, case: a pair push, then teardown with the view ref already gone. Met.
- D7, case: the operator has commit O unpushed in the prime, then a transition commit R. Owed. The operator's `git push` carries O and R, so the way forward works. Met. The landing push exists (`internal/landingshed/finalize.go:401` `pushParent`).
- D8, case 1: a pull request merged on GitHub, then Finalize's fast-forward of the prime parent. **Missed** (F1).
- D8, case 2: Finalize's merge call and the beam path where the pull request is already merged. **Missed** (F2).
- D9, case: a records-only commit after a gate pass, checked by verify and the green record. Met. `HoldsWork` after a squash: **missed** (F7).
- D10, case: a webster write under a real `_lym`. Met.
- D11, case: a board write on `lym-bolt` from `_bolt`, a linked worktree. Met.
- D12: nothing to run. Met.

**Findings**

**F1 — BLOCKING, design, D8 (with D5)**
After GitHub merges the view, Finalize's fast-forward of the parent is refused whenever the parent holds local records.
- D5 makes `FastForwardPair` commit the worktree's uncommitted records first. In the prime, batten's records are written all the time.
- That new local commit is not an ancestor of origin's merged tip. The check `IsAncestor(localHEAD, upstreamSHA)` fails and returns `FastForwardNotFastForward`. An unpushed prime records commit has the same effect.
- Finalize then halts with "the parent pair refused the fast-forward", so D8's end state, "fast-forwards the local parent", is not reached.
- The fix: bring the parent level by D7's records-only replay onto origin's tip, not by a pure fast-forward. State it in D8.
- Evidence: `internal/fabricengine/parenttips.go:309-331`, `internal/landingshed/finalize_pr.go:472-477`.

**F2 — BLOCKING, design, D8**
The view commit's identity is never pinned, and two existing head sites take the task head, not the view tip.
- `mergeOnGitHub` passes `head := TaskHead()` as the merge's expected head (`finalize_pr.go:427,448`). `openAtOnce` records `Head: TaskBranch` and `GatedHead: head` (`:511-512`). D8 renames only "Finalize's head check".
- `recordedBeamLanding` compares `status.HeadSHA` with the task head and then calls `CommitsMissing(status.HeadSHA, head)` (`:329-345`). The view is a single commit off the merge base, so every task commit reads as missing. A beam pull request that already merged is then decided again as a fresh landing.
- Re-publishing before that comparison only matches if the publish is deterministic. D8 does not say it is: a new commit date gives a new SHA on every publish.
- The fix: make the publish idempotent (reuse the existing view when its tree and parent are unchanged, or use fixed commit dates). Name the view tip as the expected head at every head site.

**F3 — MEDIUM, design, D3**
`Add` in one repository checks out the parent's run records on the new task branch.
- Today the weft fork avoids that: `createWeftWorktreeDroppingRuns` uses `worktree add --no-checkout`, and Add's first commit records the deletion of the runs root. The comment says an inherited record for the child's own slug "makes the child's seed refuse a disagreeing one".
- D3's Add writes only the origin record and the marker, so the child's seed meets the inherited runs root.
- Evidence: `internal/fabricengine/forkrecords.go:1-25`.

**F4 — MEDIUM, design, D2**
`--single-repo` pushes two refs with no stated order or recovery.
- If `lym-bolt` reaches origin and the marker push of the prime's branch is refused (for example, a protected default branch), a rerun with `--single-repo` refuses because origin holds `lym-bolt`.
- A plain one-URL clone then takes the first branch of D2 and checks out a prime whose root has no marker. D1 reads that prime as two repositories while `_bolt` sits on `lym-bolt`.
- D2's step output does not guarantee D1's input.

**F5 — MEDIUM, design, D4**
Fabric's empty records commits fail while an agent has code staged.
- `commitEmptySnapshot` calls `CommitEmpty`, which returns `ErrIndexNotEmpty` whenever `git diff --cached --quiet` reports anything staged.
- D4's path-limited rule covers only path commits. In the shared index, the tags-only and re-anchor empty commits refuse instead of landing.
- Evidence: `internal/fabricengine/recordsgit.go:180-189`, `internal/gitrepo/gitrepo.go:199-208`.

**F6 — MEDIUM, design, D4 (prime)**
The prime's records follow whatever branch the operator has checked out.
- `git switch other` in the prime swaps the live `_lym` tree, which holds batten's status for every run, for that branch's copy or for nothing.
- Later records commits then land on the other branch. In two repositories the prime's records branch is independent of the operator's checkout.
- D4 and D7 are silent on this case.

**F7 — MEDIUM, design, D9 and Notes for the plan writer**
`HoldsWork` returns false on a merge that is not clean, before it compares any tree.
- After a squash landing, the parent's committed records and the task's records both differ from the fork point. `MergeTree(parent, task)` then conflicts on records paths and returns `clean=false`.
- The note "stripping both trees is the change" prescribes a mechanism that never reaches the comparison.
- The records paths have to be stripped from the merge inputs (base, ours, theirs), not from the merged result.
- Evidence: `internal/fabricengine/parenttips.go:224-247`.

Tool calls:
1. `internal/landingshed/finalize_pr.go:470-485`. My context ended before the fast-forward after a GitHub merge, which F1 needed.
2. A grep for the parent push in `landingshed` plus the `internal/fabricengine/forkrecords.go` header. They were needed to check D7's landing-push claim and D3's fork-time records, and neither was in the manifest.
