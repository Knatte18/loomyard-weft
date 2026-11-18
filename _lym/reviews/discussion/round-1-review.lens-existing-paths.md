```yaml
lens: existing-paths
outcome: reported
```

Task: existing-paths lens review of the fabric single-repo decision record.

**Paths traced, per decision:**

- D1: `lymcwd.readRecordedAnchor` own-marker read at `weftname.SiblingPath`. Survives, because D3 moves it to the worktree root.
- D3: `post-checkout.sh` weft-sibling drift warning. Survives: with no `&lt;base&gt;-weft` directory it exits silently.
- D4: `gitrepo.StageAndCommit` (`add --`, then `commit -m -- files`). Survives.
- D4: `fabricengine.commitEmptySnapshot` → `gitrepo.CommitEmpty`. Breaks.
- D4/D6: `codetipsettle.ReturnToPreviousTransition` commit in `RecordsWorktree`. Survives.
- D5/D8: `Fabric.FastForwardPair` called from `finalize.go`, `finalize_pr.go` and `battencli/codetipsettle.go`. Breaks.
- D5/D9: `finalize.go` parent alignment, which halts when `ReadParentTips` reads `ParentAhead` or `ParentDiverged`. Breaks.
- D6: `handover.pushOwedCodeTip`, `lastTransitionCodeTip` and `deleteGivenUpCodeBranch`. Breaks.
- D8: `finalize_pr.recordedBeamLanding`'s `CommitsMissing(status.HeadSHA, head)` and `beamcli/pr.go`'s `PushedHead: head`. Breaks.
- D8: `finalize_pr.mergeOnGitHub` merge with the expected head, its pending-retry loop and its "approves again or settles the required check" halt. Breaks unless publishing is idempotent.
- D9: `websterengine.reconcileHead`, which lets only merges sit between a fork's reported head and HEAD. Breaks.
- D9: `websterengine.fixCommitRejection`. Survives, since it accepts non-merge commits.
- D9: `gatecli` `headIdentity` and `verifytree` `headTree`. These are named in D9 and survive once changed as stated.

**Findings:**

1. **BLOCKING · design · `decision-record.md` D5 (dirtiness guards) with D8 (fast-forward after a GitHub merge)**
   - Summary: D5 commits the worktree's records before `FastForwardPair` judges dirtiness. That records commit makes local HEAD something origin lacks, so `IsAncestor(localHEAD, upstreamSHA)` fails and the verb refuses `FastForwardNotFastForward`. This contradicts D5's "a fast-forward to the branch's own origin tip is unchanged" and D8's "fast-forwards the local parent to origin's parent tip".
   - The same holds before any guard runs: a parent such as the prime, carrying local records commits origin lacks, is never a fast-forward of its upstream.
   - Evidence: `internal/fabricengine/parenttips.go:309-331` (dirty check, then the `IsAncestor` refusal); callers at `internal/landingshed/finalize_pr.go:472`, `internal/landingshed/finalize.go:559` and `internal/battencli/codetipsettle.go:91`.
   - Fix: D5/D8 say how a fast-forward treats local records-only commits (for example, replaying them per D7 onto origin's tip) instead of claiming the fast-forward is unchanged.

2. **BLOCKING · design · `decision-record.md` D9 (code identity) and D5**
   - Summary: Finalize's parent alignment halts on `ParentAhead` or `ParentDiverged` with "push or reconcile". In a one-repository hub every parent carrying unpushed records commits (the prime's batten records, a beam's own records) reads ahead, so every landing halts.
   - D9 classifies HEAD-unchanged, HEAD-moved and ancestry readers, but not a local-vs-upstream tip relation. The record is silent on this reader kind.
   - Evidence: `internal/landingshed/finalize.go:547-553`; `ReadParentTips` at `internal/fabricengine/parenttips.go:47`.

3. **BLOCKING · design · `decision-record.md` D6 (hand-over) with D4 (`Warp-SHA` = HEAD before the commit)**
   - Summary: the owed-tip hand-over reads the code tip from the last transition's `Warp-SHA` and pushes it to the code branch on origin. In one repository the code branch is the records branch, and that tip is an ancestor of the transition commit already on origin.
   - Two outcomes follow:
     - `move(originTip, tip)` rewinds origin's branch off the transition records.
     - On a given-up tip, `deleteGivenUpCodeBranch` deletes the single branch, records included.
   - D6 keeps "hands over … as today" and says nothing about this path.
   - Evidence: `internal/fabricengine/handover.go:203-213`, `:274-311`, `:315-333`, `:341-348`.
   - Fix: D6 states that `pushOwedCodeTip`'s owed-tip push does not run in the one-repository layout, since one push carries both.

4. **MEDIUM · design · `decision-record.md` D4 (records commit on the code branch)**
   - Summary: D4 makes records commits path-limited so agent-staged code is never swept in. Fabric's empty records snapshot commit is not path-limited: `CommitEmpty` refuses with `ErrIndexNotEmpty` whenever the shared index holds anything staged.
   - So any agent-staged code fails every empty snapshot or re-anchor commit in a one-repository worktree.
   - Evidence: `internal/fabricengine/recordsgit.go:179-189`; `internal/gitrepo/gitrepo.go:199-210`.

5. **MEDIUM · design · `decision-record.md` D9 (SHA readers)**
   - Summary: webster's `reconcileHead` tolerates only merge commits between a fork's reported `head_sha` and HEAD. A records-only commit landing on the same branch during a batch is a non-merge commit, so webster refuses the batch. Such commits come from webster's own sync through `Fabric.Commit`, a transition, or `lym fabric sync`.
   - D9's reader kinds do not cover a "only merges may sit between" walk.
   - Evidence: `internal/websterengine/gitwrap.go:113-146`, refusal at `:132-136`.

6. **MEDIUM · design · `decision-record.md` D8 (records-free view)**
   - Summary: existing pull-request paths compare the pull request's head with the task head. With a view head they misread:
     - `recordedBeamLanding` runs `CommitsMissing(status.HeadSHA, head)`. A merged view commit, parented on the merge base, lacks every task commit, so an already-merged beam pull request re-resolves as a no-pull-request landing.
     - `beamcli`'s `PushedHead: head` records the task head.
   - D8 names only Finalize's head check.
   - Evidence: `internal/landingshed/finalize_pr.go:329-347`; `internal/beamcli/pr.go:108`, `:196`, `:346`.

7. **MEDIUM · design · `decision-record.md` D8 (publish before merging)**
   - Summary: D8 publishes a view before every open, update or merge, and does not say publishing is idempotent. A view commit rebuilt from the same tree and parent still gets a new SHA from its commit time.
   - The fresh head published right before `mergeOnGitHub`'s merge resets GitHub's required checks and dismisses stale approvals. The merge then refuses as pending until `prMergeRetryWindow` ends, or halts with "approves again or settles the required check". A resume publishes again and repeats the cycle.
   - Evidence: `internal/landingshed/finalize_pr.go:442-466` (merge with the expected head, the pending retry, the halt), `:544` (the refusal reason).

Tool calls:

1. `handover.go:270-333` and `post-checkout.sh`: the owed-tip push and delete path, and the checkout hook, were outside the manifest.
2. `gitwrap.go` `reconcileHead` and `fixCommitRejection` bodies, plus the `FastForwardPair` callers: the manifest listed only signatures.
3. `handover.go` function list, `finalize.go:545-575` and `battencli/codetipsettle.go:75-100`: how the owed-tip push is entered, and the parent-alignment and fast-forward callers.
