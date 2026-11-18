```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I attacked each bound, guard, ordering promise and claim about existing code in the decision record.

**Claims attacked**

- D1, the marker sits only at the weft root in a two-repository hub: held for code worktrees. I did not verify the legacy Bolt marker fallback (`internal/lymcwd/anchor.go:192`).
- D2, `--single-repo` creates only and refuses `lym-bolt`, a weft binding or a second URL: held for those inputs. Broken on a protected prime branch (F2).
- D2, a single URL whose origin lacks `lym-bolt` stays a weft URL: held. The weft's Bolt branch is `main`, not `lym-bolt`.
- D4, the path-limited commit never sweeps in staged code: held for `StageAndCommit` (`internal/gitrepo/gitrepo.go:141-160`). Broken for fabric's empty records commits (F5).
- D4, one write lock plus a bounded `index.lock` retry: held. An agent's own git takes no fabric lock, and the retry covers that.
- D4, an agent's `git add -A` is tolerated because the strip is by path: held only while the marker the strip reads is trusted (F6).
- D4, `Warp-SHA` names HEAD before the commit, and `OriginCodeTip` reads an ancestor: held. `ReturnToPreviousTransition` goes unaddressed (F9).
- D5, a records write between step 1 and step 2 reads as a dirty refusal: held as stated. Broken for a write between step 2 and step 3 (F7).
- D5, each side's records paths come from its own marker: broken (F6).
- D5, fabric's dirtiness guards commit records first and then judge code only: broken for `FastForwardPair` (F3).
- D5, a fast-forward to the branch's own origin tip is unchanged: broken when local records commits are unpushed (F3).
- D6, push once under one lock with `PushOwed` read as today: held.
- D6, teardown tolerates a view ref already gone: held. A post-merge push recreating the view is cleaned up by teardown.
- D7, fabric rewrites only its own records-only commits and never a code commit: broken, because the predicate is undefined and forgeable (F4).
- D7, the prime push guard admits records-only commits and commits carrying the merge trailer: broken by a forged trailer and by second-parent commits (F4), and by a protected branch (F2).
- D7, an owed prime push names `git push` as the way forward: broken on a protected branch (F2).
- D8, the verb force-pushes only `lym-view/` refs: held.
- D8, publish before merging and compare against the published tip: broken (F1).
- D8, the view's parent comes from the origin record's `parent_branch`: broken when that branch differs from the pull request's base (F8).
- D8, after a GitHub merge Finalize fast-forwards the local parent: broken (F3).
- D8, concurrent publishes under the shared lock: held for the merge, since the merge passes the expected head (`internal/landingshed/finalize_pr.go:448`). PR-Review can still read a stale view (F10).
- D9, nothing changes in a two-repository hub: held. This repository tracks no `_lym` or `.lym-anchor`.
- D9, changes under records paths pass gates unseen: held, bounded by the strip at landing, given F6.

**Findings**

**F1 — BLOCKING · design · D8**
Publishing right before a merge makes a new head every time, so required checks and approvals reset and the merge cannot pass.
- The view commit's SHA carries its commit date, and D8 states no idempotent reuse when the tree and parent are unchanged.
- PR-Review publishes, and then Finalize publishes again before `Merge`.
- GitHub reruns required checks on the new head and, under stale-review dismissal, drops approvals.
- `mergeOnGitHub` retries a pending refusal only within `prMergeRetryWindow`, then halts with "approves … again or settles the required check" (`internal/landingshed/finalize_pr.go:441-466`, `:544`).
- `lym loom resume` publishes again and loops.
- Fix: the record states that the publish reuses the existing view when its tree and parent match, or builds a deterministic commit.

**F2 — BLOCKING · design · D7 and D2**
A prime branch that refuses direct pushes blocks every prime records push forever, and the stated way forward fails too.
- Batten's status is under the prime's anchor, and a transition push is synchronous; an owed push holds the run (PATTERN-pair-remote).
- In one repository those pushes go to the operator's code branch. On a protected branch the refusal is permanent, and the notice's `git push` is refused as well.
- Protected-branch refusals already happen: `protectedBranchRefusal` (`internal/landingshed/finalize.go:844-851`).
- `--single-repo`'s push of the marker commit to the prime's branch fails the same way, after its creation checks.
- A rerun then refuses, because origin now holds `lym-bolt`.
- Neither D2 nor D7 bounds this.
- It breaks PATTERN-refusal-way-forward: the named way forward is not reachable.

**F3 — BLOCKING · design · D5 and D8**
Committing records first turns every fast-forward of a live worktree into a not-fast-forward refusal.
- `FastForwardPair` refuses unless `IsAncestor(localHEAD, upstream)` (`internal/fabricengine/parenttips.go:326-331`).
- D5 makes its dirtiness guard commit the worktree's records first. The local branch is then ahead of origin, so the fast-forward always refuses until that commit is pushed.
- The prime always holds live records. After a GitHub merge, D8's "fast-forwards the local parent to origin's parent tip" therefore fails.
- That fast-forward is today's `mergeOnGitHub` step (`internal/landingshed/finalize_pr.go:413-415`).
- Unpushed operator commits on the prime break it too.
- The record does not say whether to push first or to replay records onto origin's tip.

**F4 — BLOCKING · design · D7**
The predicates the replay bound and the prime guard rest on are undefined, and as fabric reads trailers today they are forgeable.
- D7 does not define "records-only commit fabric made", "carries fabric's merge trailer" or "every commit origin lacks".
- Fabric matches a trailer key on any message line: `parseWarpSHATrailer` (`internal/fabricengine/trailer.go:145-158`) and `trailerValue` (`internal/fabricengine/handover.go:366-374`).
- An agent's code commit whose message holds `Warp-SHA: x`, or the merge trailer, would be replayed or admitted.
- A non-squash local landing's second-parent task commits are neither records-only nor trailered. Whether they count as "lacking" depends on `--not origin/&lt;branch&gt;` versus `--not --remotes`.
- Fix: define each predicate as a git query, for example records-only by changed paths and a trailer. State the bound against a forged trailer.

**F5 — MEDIUM · design · D4**
Fabric's empty records commits fail whenever an agent has staged code.
- `commitEmptySnapshot` calls `CommitEmpty` (`internal/fabricengine/recordsgit.go:180-181`).
- `CommitEmpty` returns `ErrIndexNotEmpty` when `diff --cached` shows anything (`internal/gitrepo/gitrepo.go:199-210`).
- In one repository the shared index holds the agent's staged code, so the re-anchor and snapshot empty commits fail.
- D4's path-limited rule covers only path commits.

**F6 — MEDIUM · design · D5 and D9**
A marker edited on the source side makes the strip miss the real records directory, which then merges into the parent.
- D5 reads each side's records paths from its own `.lym-anchor`.
- A task branch whose marker names anchor `X` has `X/_lym` stripped, while the real `_lym` reads as code. It merges into the parent, and D9's code identity counts it as code.
- A change of the marker outside an anchor-change records commit is neither refused nor bounded.

**F7 — MEDIUM · design · D5**
A live records write between step 2 and step 3 is lost without a trace.
- After `merge --no-commit`, step 3 restores source-changed records paths in the worktree from the receiving HEAD.
- A receiving-side process, such as a prime's batten status writer, that writes such a path in that window is overwritten.
- The stated bound covers only the window between step 1 and step 2. Merges take the weft write lock (`internal/fabricengine/merge.go:31-38`); D5 does not state that records writers take it.

**F8 — MEDIUM · design · D8**
A view built against the origin record's `parent_branch` can differ from the pull request's base.
- `lym beam pr` opens against `side.ParentBranch` (`internal/beamcli/pr.go:36-42`, `:90`).
- When that branch differs from the origin record's, the view's parent and its records come from another merge base. The pull request's merge then changes the real parent's records and carries unrelated history as one commit.
- Root pairs record an empty `parent_branch` (`internal/fabricengine/add.go:429-438`), so the view has no parent at all.
- D8 does not require the view base to equal the pull request's base.

**F9 — LOW · design · D4 and D6**
`ReturnToPreviousTransition` has no defined meaning in one repository.
- It "gives up the code tip" by committing the previous transition's status on the records branch (`internal/fabricengine/codetipsettle.go:30-40`).
- With one branch, that commit lands on top of the given-up tip, so the code stays.
- The record is silent on whether the path is dead or must change.

**F10 — LOW · design · D8**
A concurrent publish can leave an older view on GitHub.
- The publish runs under the shared teardown lock, so two publishes can race, for example a push's refresh and PR-Review's publish. The last force-push wins and can install an older tree.
- Finalize's expected-head merge catches this; PR-Review can review a stale head.
- D8 states no ordering.

**Tool calls**

1. Grepped landingshed for the parent push and fabricengine for the prime's records-only push. My context lacked whether Finalize pushes the parent and how prime pushes are classified.
2. Read `internal/landingshed/finalize.go:810-870`. My context lacked `pushParent` and its protected-branch refusal handling.
