---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:141
    summary: "D8 publishes a view before every open, update and merge but never says the publish is idempotent, so each publish makes a new view SHA, which resets GitHub's required checks and stale approvals right before Finalize merges, and a resume repeats the cycle"
    fix: "In D8, state that the publish reuses the existing `lym-view/<branch>` tip when its tree and its parent equal the ones it would write, and otherwise writes a commit with fixed author and committer identity and dates so the same inputs give the same SHA. Add to the D8 acceptance criterion that two publishes of an unchanged branch leave the view ref on the same SHA and push nothing the second time."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:98
    summary: "Unpushed local records commits make every parent read as ahead of its upstream, so Finalize's parent alignment halts on ParentAhead or ParentDiverged and FastForwardPair refuses as not a fast-forward, which breaks D5's claim that a fast-forward to origin is unchanged and D8's fast-forward of the parent after a GitHub merge"
    fix: "In D5, replace the sentence that a fast-forward to the branch's own origin tip is unchanged with the rule that bringing a branch level with its origin tip first commits the worktree's records, then replays the local records-only commits onto origin's tip as D7 replays them, and refuses only when a local non-records commit is unpushed. In D8, say that after GitHub merges the view Finalize brings the local parent level by that rule instead of a pure fast-forward. In D9, add the local-versus-upstream tip relation (`fabricengine.Fabric.ReadParentTips`, Finalize's parent alignment) as a reader kind that judges code commits only. Add an acceptance row: with uncommitted and unpushed parent records, a pull-request landing and a local landing both complete."
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:125
    summary: "D7 and D2 push records to the prime's branch, the operator's code branch, with no bound for a branch that refuses direct pushes, so every prime records push is owed for good, every run held at a prime transition waits for ever, the named way forward git push is refused too, and --single-repo fails half-done"
    fix: "Add to D7 a decision for a prime branch whose origin refuses a direct push (lym already recognises the refusal through `protectedBranchRefusal` in `internal/landingshed/finalize.go`): either `--single-repo` and the one-URL clone refuse such a hub at clone time, naming the protection as the cause, or the prime's records go somewhere a push is accepted. State which, and the way forward the refusal names. Add a D2 bound and an acceptance row for a protected prime branch."
  - id: F4
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:121
    summary: "D7's bound that fabric never rewrites a code commit, and its prime push guard, rest on three predicates the record never defines (a records-only commit fabric made, a commit carrying fabric's merge trailer, a commit origin lacks), and fabric reads trailers from any message line, so a forged trailer can pass the replay or the guard"
    fix: "In D7, define each predicate as the git read that decides it. A records-only commit is a non-merge commit whose changed paths all lie under the records paths of both its parent and itself, whatever trailer it carries. A commit carries the merge trailer only when it is a merge commit fabric wrote. Pick and state either that a merge's second-parent commits count as lacking or that lacking means not reachable from any origin ref. State the bound: a commit message alone never admits a commit that changes code paths."
  - id: F5
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:112
    summary: "D6 keeps the hand-over as today, but the owed-code-tip paths (pushOwedCodeTip, deleteGivenUpCodeBranch, ReturnToPreviousTransition) act on the code branch separately from the records branch; with one branch they rewind origin off the transition records or delete the branch that holds them"
    fix: "In D6, state that in a one-repository hub no code tip can be owed apart from its records, because one push carries both. So the owed-code-tip push, the given-up code branch delete and the return to a previous transition do not run there. A rejected push or a takeover goes through D7's replay and today's hand-over only. Add an acceptance row: after a takeover in a one-repository hub, origin's branch keeps every transition commit."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:142
    summary: "D8 moves only Finalize's head check to the view tip, while mergeOnGitHub's expected head, openAtOnce's recorded head and gated head, recordedBeamLanding's CommitsMissing check and lym beam pr's PushedHead still take the task head, so a merged view reads as lacking every task commit"
    fix: "In D8, replace the sentence about Finalize's head check with a rule: every place that compares, records or passes a pull request's head uses the view tip the publish returned. A merged pull request counts as holding the task's work when its head is the view published from the task tip, or when D9's held-work check reads true. Do not use commit ancestry."
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:76
    summary: "D4 makes records commits path-limited, but fabric's empty records commits go through gitrepo CommitEmpty, which refuses whenever anything is staged, so an agent's staged code fails every snapshot and re-anchor commit in a one-repository worktree"
    fix: "In D4, add a bullet that fabric's empty records commits (`commitEmptySnapshot` in `internal/fabricengine/recordsgit.go`) land with the agent's staged entries left staged and none of them committed, as the path-limited commit does. Add an acceptance row for an empty records commit made while code is staged."
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:160
    summary: "D9's SHA-reader kinds miss webster's reconcileHead, which lets only merge commits sit between a fork's reported head and HEAD, so a records-only commit landing during a batch makes webster refuse the batch"
    fix: "In D9's SHA-readers bullet, add the kind that walks commits between two SHAs and allows only certain commits (webster's `reconcileHead` in `internal/websterengine/gitwrap.go`). Records-only commits, as D7 defines them, pass such a walk like a clean parent merge does. Add an acceptance row: a records commit made during a webster batch does not refuse the batch."
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:95
    summary: "D5, D7 and D9 read a commit's records paths from that one commit's marker, so an anchor change makes the old anchor's _lym deletions read as code, and a source-side marker edited to another anchor makes the strip miss the real _lym, which then merges into the parent as code"
    fix: "In D5, state that the records paths of a commit or merge side are the union of the anchors its own marker and its parent's marker name. Also state that a merge whose source changed `.lym-anchor` other than through fabric's anchor-change commit refuses, naming the marker change. Apply the same union in D7's records-only test and D9's code identity. Add acceptance rows for an anchor-change commit and for a source with an edited marker."
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:103
    summary: "D5's bound covers a records write between steps 1 and 2 only; a receiving-side write between step 2 and step 3 to a path the source changed is silently overwritten when step 3 restores the receiving HEAD"
    fix: "In D5, state either that records writers in the receiving worktree are held off from step 1 to step 4 by the merge's write lock, naming the lock they take, or that step 3 compares each restored path against what step 2 wrote and refuses the merge, as today's dirty refusal does, when a path changed in between. Extend the Bound line to cover the window between step 2 and step 3."
  - id: F11
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:63
    summary: "D3's Add creates the task worktree on a branch forked from the parent, so it checks out the parent's committed run records, which today's weft fork drops so that the child's seed does not meet an inherited record for its own slug"
    fix: "In D3's Add bullet, state that the records commit Add makes on the new branch also deletes the parent's run-records root, and that the root is never written to the new worktree's disk, as `createWeftWorktreeDroppingRuns` in `internal/fabricengine/forkrecords.go` does today. Add to the D3 acceptance criterion that a task forked from a parent carrying run records starts with none."
  - id: F12
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:74
    summary: "The prime's live records, batten's status for every run among them, sit on whatever branch the operator has checked out, so a branch switch, reset or stash in the prime swaps or drops them, and D4 and D7 are silent on it"
    fix: "Add a decision on the prime's records branch. Either fabric refuses to commit records while the prime is not on the branch the hub was cloned on, recording that branch, and names switching back as the way forward. Or state the hazard as a bound in D7 with the docs warning D2 already requires. Add an acceptance row for the chosen behaviour."
  - id: F13
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:229
    summary: "The plan-writer note says stripping both trees is the HoldsWork change, but HoldsWork returns false on an unclean merge before it compares trees, and after a strip landing the records paths conflict, so stripping the result never reaches the comparison"
    fix: "Rewrite the note to say that the held-work checks strip the records paths from the merge base and from both sides before the in-memory merge, so records conflicts cannot make the merge unclean. Then compare the code trees. Add to the D9 acceptance criterion that `HoldsWork` reads true after a squash landing whose parent and task records both changed since the fork."
  - id: F14
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:48
    summary: "D2's --single-repo pushes lym-bolt and the prime's marker commit with no stated order or recovery, so a refused marker push leaves an origin holding lym-bolt and no marker, which a rerun refuses and a one-URL clone then reads through D1 as a two-repository prime"
    fix: "In D2, state that `--single-repo` pushes the prime's marker commit before it creates `lym-bolt` on origin, and pushes nothing further when that push is refused. Also state that the one-URL clone of an origin holding `lym-bolt` refuses when the prime's branch lacks `.lym-anchor` at its root, naming the missing marker. Add both to the D2 acceptance criterion."
  - id: F15
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:76
    summary: "The Decisions that rest on git and GitHub behaviour state that behaviour without its observed result or pin, so every round has to repeat the experiment"
    fix: "For each tool fact, record the observed result and its pin: a test, a recorded command with its output, or the tool's documentation. Do this in the Decision or in a Constraint. The facts are: D4, that `git commit -- <paths>` commits only those paths and leaves other staged entries staged (`git-commit` documentation, `--only`, plus `gitrepo.StageAndCommit`'s test); D5, that `git merge --no-commit --no-ff`, followed by restoring paths from HEAD in index and worktree and removing source-only paths, resolves conflicts on those paths; D8, that a GitHub pull request whose head is a commit parented on the merge base shows and merges only the view's changes; D6, that GitHub deletes a merged head branch only under the repository's auto-delete setting."
  - id: F16
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md:202
    summary: "The board entry says the operator wants this mode taken in if fabricengine is rewritten (note clean-room-module-rewrite), while the record builds it into the current package now; whether to build now or only spec it for the rewrite is open for the parent (support log Q6)"
    fix: "The parent decides whether the run builds the mode into the current `internal/fabricengine` now or writes only a spec for the rewrite, and records the call with `lym loom decision add`."
  - id: F17
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:191
    summary: "The Constraints keep every touched PATTERN overview line, but PATTERN-cwd-resolution's overview reads the anchor from the worktree's weft-root .lym-anchor, which a one-repository worktree does not have"
    fix: "Under `## PATTERN changes`, add a `changes` bullet for `PATTERN-cwd-resolution` whose new text reads the anchor from the worktree's own `.lym-anchor`, at its records root (the weft root in a two-repository hub, the worktree root in a one-repository hub). Drop `PATTERN-cwd-resolution` from the Constraint's list of overview lines that stay."
  - id: F18
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:121
    summary: "D7 rewrites a replayed transition commit's Warp-SHA to its new parent even when origin's diverging commits carry code, so the transition then names a code tip it never judged, which the hand-over and OriginCodeTip read"
    fix: "In D7, state that a replay rewrites trailers only when every commit origin added is records-only. Otherwise it stops as owed, naming origin's code commits, as it does for local code commits."
  - id: F19
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:119
    summary: "D7 applies its records-only replay to every replay on any branch, which read literally covers lym-bolt, whose board commits touch no records path and which D11 keeps on today's by-entry Bolt replay"
    fix: "In D7, change on any branch to on any code branch, and add that `lym-bolt` keeps today's by-entry Bolt replay."
  - id: F20
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:126
    summary: "D7 says a run in the prime waits at its transition on an owed prime push, but does not name the failed-push class or runs the guard's refusal records, and the act rule counts only certain classes whose runs name the slug"
    fix: "In D7, name the failed-push class the prime guard's refusal records, one the act rule already reads as owed, and state that its runs list the prime runs whose transition the push carries."
  - id: F21
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:140
    summary: "D8 takes the view's parent from the origin record's parent_branch without requiring it to equal the pull request's base branch, so a mismatch makes the merged view carry another merge base's records into the real parent"
    fix: "In D8, state that the view is built against the branch the pull request targets, and that the publish refuses when that branch differs from the origin record's `parent_branch`, naming both."
  - id: F22
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:198
    summary: "Auto-mode assumptions call the mode built now auto-picked, while the support log's Q6 and Open risks leave that open for the parent, and the list omits the auto-picks the log records for D5, D7, D9 and D11"
    fix: "In `## Auto-mode assumptions`, drop and the mode built now from the Scope Out line. Add one line each for D5 (strip at every merge), D7 (records-only replay and the guarded prime push), D9 (code identity in both layouts) and D11 (Bolt on orphan `lym-bolt`), each ending auto-picked."
  - id: F23
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:211
    summary: "The acceptance criteria give no checkable behaviour for D4's single write lock over records and code commits or for D7's hold of a prime run at its transition while the prime push is owed"
    fix: "Extend the D4 criterion: a fabric records commit and a fabric code commit started together in one worktree run one after the other. Extend the D7 criterion: a run in the prime with an owed prime push does not act past its transition until the push lands."
---

# Discussion review — round 1

The record designs the one-repository layout coherently at the level of which side owns what. The breaks are where its mechanisms meet code that already runs: the parent fast-forward and alignment, the pull-request head sites, the hand-over's code-tip paths and protected prime branches. D7's predicates are also undefined.
Every fork finding below was checked against the code it cites.

### [BLOCKING] D8's publish is not idempotent

**Location:** `_lym/discussion/decision-record.md:141`
**Issue:** D8 says PR-Review, Finalize and `lym beam pr` publish before opening, updating or merging a pull request.
A view commit rebuilt from the same tree and parent still gets a new SHA from its commit date.
Finalize's merge passes the head as the expected head and retries a pending refusal only within `prMergeRetryWindow` (`internal/landingshed/finalize_pr.go:441-466`).
Its refusal reason already names re-approval and required checks (`:544`).
A fresh head reruns the checks and can dismiss approvals, so the merge halts, and a resume publishes again.

### [BLOCKING] Local records commits make every parent read as ahead of origin

**Location:** `_lym/discussion/decision-record.md:98`
**Issue:** Finalize's parent alignment halts on `ParentAhead` and `ParentDiverged` (`internal/landingshed/finalize.go:547-553`).
`FastForwardPair` refuses unless the local HEAD is an ancestor of the upstream tip (`internal/fabricengine/parenttips.go:326-331`).
D5 has the dirtiness guards commit records first, and the prime and every beam always carry live records.
So the parent is ahead whenever its records have not been pushed yet.
D8's fast-forward after a GitHub merge (`internal/landingshed/finalize_pr.go:472-477`) and D5's "unchanged" fast-forward then refuse.

### [BLOCKING] A prime branch that refuses direct pushes has no bound

**Location:** `_lym/discussion/decision-record.md:125`
**Issue:** In one repository the prime's records pushes go to the operator's code branch.
Batten's status lives under the prime's anchor, and an owed transition push holds the run (`PATTERN-pair-remote`).
lym already meets protected parents (`protectedBranchRefusal`, `internal/landingshed/finalize.go:844-851`).
On such a branch the push is refused for good, and so is the `git push` D7 names as the way forward.
That way forward cannot be reached, contrary to `PATTERN-refusal-way-forward`.
D2's `--single-repo` push of the marker commit fails the same way.

### [BLOCKING] D7's predicates are undefined and forgeable

**Location:** `_lym/discussion/decision-record.md:121`
**Issue:** The replay bound ("never a code commit") and the prime guard rest on three predicates: a records-only commit fabric made, a commit carrying fabric's merge trailer, and a commit origin lacks.
The record defines none of them.
Fabric reads a trailer from any message line (`parseWarpSHATrailer`, `internal/fabricengine/trailer.go:145-158`; `trailerValue`, `internal/fabricengine/handover.go:366-374`), so any commit author can forge one.
A non-squash landing's second-parent commits fall under neither admitted kind.
The attack-surface rule asks for a stated bound on a guard that can be bypassed this way.

### [BLOCKING] D6's code-tip hand-over paths act on the records branch

**Location:** `_lym/discussion/decision-record.md:112`
**Issue:** The hand-over reads the code tip from the last transition's `Warp-SHA` and pushes or deletes the code branch on origin (`internal/fabricengine/handover.go:315-348`).
`ReturnToPreviousTransition` gives a code tip up by committing on the records branch (`internal/fabricengine/codetipsettle.go:30-40`).
With one branch, the push rewinds origin off the transition records, and the delete removes the branch that holds them.
The return commits on top of the tip it means to give up.
D6 says only "as today".

### [MEDIUM] Pull-request head sites still take the task head

**Location:** `_lym/discussion/decision-record.md:142`
**Issue:** `mergeOnGitHub` (`finalize_pr.go:427,448`), `openAtOnce` (`:511-512`), `recordedBeamLanding`'s `CommitsMissing` (`:329-345`) and `internal/beamcli/pr.go`'s `PushedHead` all use the task head.
A view parented on the merge base lacks every task commit, so an already merged beam pull request is decided again.

### [MEDIUM] Empty records commits refuse with code staged

**Location:** `_lym/discussion/decision-record.md:76`
**Issue:** `commitEmptySnapshot` (`internal/fabricengine/recordsgit.go:180-189`) calls `CommitEmpty`.
`CommitEmpty` returns `ErrIndexNotEmpty` when anything is staged (`internal/gitrepo/gitrepo.go:199-210`).
In the shared index, that is any agent's staged code.

### [MEDIUM] D9 misses webster's merges-only walk

**Location:** `_lym/discussion/decision-record.md:160`
**Issue:** `reconcileHead` (`internal/websterengine/gitwrap.go:113-146`) refuses when a non-merge commit sits between a fork's reported head and HEAD.
A records commit on the code branch is such a commit.

### [MEDIUM] Records paths come from one marker

**Location:** `_lym/discussion/decision-record.md:95`
**Issue:** `committedMarker` and `treeAnchor` (`internal/fabricengine/anchormarker.go:93-118`) read one marker per revision.
An anchor-change commit then shows its old-anchor deletions as code.
A source-side marker edited to another anchor makes D5's strip miss the real `_lym`, which merges into the parent, and D9 counts it as code.
Apart from that edit, a change under `_lym` or `.lym-anchor` that passes gates unseen never reaches the parent, since D5 and D8 strip it.

### [MEDIUM] A records write between merge steps 2 and 3 is lost

**Location:** `_lym/discussion/decision-record.md:103`
**Issue:** Step 3 restores source-changed records paths from the receiving HEAD.
A receiving-side writer that writes such a path after step 2 is overwritten, and the Bound line covers only the window between steps 1 and 2.

### [MEDIUM] Add inherits the parent's run records

**Location:** `_lym/discussion/decision-record.md:63`
**Issue:** Today `createWeftWorktreeDroppingRuns` (`internal/fabricengine/forkrecords.go:1-25`) keeps the parent's run-records root out of a new pair, because an inherited record for the child's slug makes its seed refuse.
D3's Add forks the code branch, records included, and writes only the origin record and the marker.

### [MEDIUM] The prime's records follow the operator's checkout

**Location:** `_lym/discussion/decision-record.md:74`
**Issue:** In a two-repository hub the prime's `_lym` is a junction into a weft branch the operator never switches.
In one repository it sits on the operator's checked-out branch.
A `git switch`, `reset` or `stash` there swaps or drops batten's live status for every run.

### [MEDIUM] The held-work note prescribes a mechanism that never reaches its comparison

**Location:** `_lym/discussion/decision-record.md:229`
**Issue:** `HoldsWork` returns false on an unclean in-memory merge before it compares trees (`internal/fabricengine/parenttips.go:224-247`).
After a strip landing, both sides' records have changed since the fork, so the merge conflicts on them.

### [MEDIUM] --single-repo can leave origin half-made

**Location:** `_lym/discussion/decision-record.md:48`
**Issue:** D2 says only "pushes both".
Suppose `lym-bolt` lands and the marker push is refused.
A rerun with `--single-repo` then refuses because `lym-bolt` exists.
A one-URL clone checks out a prime with no marker, which `readRecordedAnchor` (`internal/lymcwd/anchor.go:176-190`) silently reads as two repositories.

### [MEDIUM] Tool facts carry no pin

**Location:** `_lym/discussion/decision-record.md:76`
**Issue:** D4's `git commit -- <paths>` behaviour, D5's merge-then-restore sequence, D8's pull request on a merge-base-parented view and D6's GitHub head deletion are each stated without an observed result or a pin.
`gitrepo.StageAndCommit` (`internal/gitrepo/gitrepo.go:141-160`) already runs D4's form and can serve as its pin.

### [MEDIUM] Build now or only for the rewrite is the parent's call

**Location:** `_lym/discussion/decision-record.md:202`
**Issue:** The board entry ties this mode to a fabricengine rewrite (note `clean-room-module-rewrite`), and no `Decision for fabric-single-repo-mode:` note exists.
The writer recorded the question under Open risks and left Q6 open, so it goes to the parent.

### [MEDIUM] PATTERN-cwd-resolution's overview names the weft-root marker

**Location:** `_lym/discussion/decision-record.md:191`
**Issue:** The overview line reads the anchor "from the worktree's weft-root `.lym-anchor`".
The Constraint keeps that line while D1 and D3 read the marker at a one-repository worktree's root.
The declared PATTERN changes list only `PATTERN-single-repo-hub`, so the overview and the design disagree.

### [LOW] Replay rewrites Warp-SHA over foreign code

**Location:** `_lym/discussion/decision-record.md:121`
**Issue:** `lastTransitionCodeTip` (`internal/fabricengine/handover.go:336-346`) takes a transition's code tip from its `Warp-SHA`.
A rewrite over code origin added makes that trailer name a tip the transition never judged.

### [LOW] D7's any-branch scope covers lym-bolt

**Location:** `_lym/discussion/decision-record.md:119`
**Issue:** The Bolt is replayed by entry (`internal/fabricengine/replay.go:4-5`).
D7's wording would put it under a records-only rule that no board commit meets.

### [LOW] The owed prime push names no failed-push class

**Location:** `_lym/discussion/decision-record.md:126`
**Issue:** `FailedPushesOwedBy` (`internal/fabricengine/failedpush.go:272-293`) holds a run only for certain classes whose runs name its slug.
D7 names neither the class nor the runs.

### [LOW] The view base is not tied to the pull request's base

**Location:** `_lym/discussion/decision-record.md:140`
**Issue:** `lym beam pr` opens against its side's parent branch (`internal/beamcli/pr.go:90`).
D8 builds the view from the origin record's `parent_branch` and never requires the two to match.

### [LOW] Auto-mode assumptions disagree with the log

**Location:** `_lym/discussion/decision-record.md:198`
**Issue:** The list calls building now an auto-pick, while Q6 is open.
It also omits the auto-picks the log's turns 4 to 6 record.

### [LOW] Acceptance criteria miss two stated behaviours

**Location:** `_lym/discussion/decision-record.md:211`
**Issue:** D4's single write lock and D7's hold of a prime run have no criterion a test can check.
D6's owed-push record has one.

## Rejected

- bound-breaker F10, a concurrent publish installing an older view: both publishers build from the same local branch in one worktree, and Finalize's expected-head merge refuses a stale head, so the race costs at most one stale view read by PR-Review. F1's idempotent publish narrows it further.
- bound-breaker F9 and existing-paths 3 are one defect, kept as the D6 finding.
- The fast-forward findings of interactions F1, walkthrough F1, bound-breaker F3 and existing-paths 1 and 2 are one defect, kept as the parent-ahead finding.
- Focus, D2 and D6 bounds: the wrong-repo `--single-repo` bound is stated and documented, and a view ref deleted by someone else is tolerated at teardown and recreated by the next publish. Neither is a defect.
- Focus, relocation: D2's docs sentence, D6's mechanics and the Constraints' PATTERN-background paragraph each carry a requirement the plan writer needs, and D1's Rationale is allowed beside its Decision. None is moved.
