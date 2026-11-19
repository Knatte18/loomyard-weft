# Fabric: run a hub as one repository, records on each code branch

A hub can now run as one repository instead of two.
The records that a two-repository hub keeps in a sibling weft checkout sit under `<anchor>/_lym` on each code branch,
and the board lives on an orphan branch `lym-bolt`.
The two-repository layout is unchanged.

## What changes

- **Layout.**
  A worktree is part of a one-repository hub when `.lym-anchor` sits at its own root.
  `lymcwd` reads the marker from the worktree, with `_bolt` or a weft sibling missing too, and exports the predicate;
  `Resolve` and `ResolveWithin` use it.
- **Clone.**
  `lym fabric clone <url>` over an origin holding `lym-bolt` clones the prime and adds `_bolt` as a worktree on `lym-bolt`.
  `lym fabric clone --single-repo <url>` over a fresh origin commits the marker and seed records on the prime's branch,
  pushes that commit, then creates `lym-bolt` as an orphan branch.
  A refused marker push leaves no `lym-bolt` on origin and removes the hub directory;
  a protected-branch refusal names the protection.
  `--single-repo` refuses an origin that holds `lym-bolt` or a weft binding, and a second URL.
  A one-URL clone whose prime branch lacks `.lym-anchor` refuses and names the marker.
- **Add and wiring.**
  `Add` makes one worktree on one branch and commits the origin record and marker as a records commit.
  That commit drops the parent's run records.
  Junction wiring, junction repair, `.lym` adoption and unwire do not run.
  The shared exclude gets `.lym`, the `.scratch` link and heddle's rendered files, never `_lym` or `.lym-anchor`.
- **Records commits.**
  A records commit is path-limited, so code an agent staged stays staged.
  It takes the worktree's write lock, retries a held `index.lock` for a bounded time, and names the prior HEAD in `Warp-SHA`.
  During a merge, cherry-pick or rebase it no longer skips:
  mid-merge it writes the commit from a temporary index and moves HEAD onto it,
  and `statuscommit` waits at the transition, with a warning naming the operation, for an operation it cannot route around.
- **Merges.**
  Every fabric verb that brings one branch into another (merge, squash, merge-in, beam) keeps the receiving side's records:
  it commits the receiver's records, merges without committing, then resets the records paths to the receiver's HEAD.
  A fast-forward onto another branch's commits is never made; the verb makes a merge commit with a fabric merge trailer.
  A merge whose source changed `.lym-anchor` outside an anchor-change commit refuses.
  `lym fabric merge --abort` and `--continue` handle a merge stopped before the records step.
- **Push and teardown.**
  A pair push pushes the one branch once under one push lock.
  The owed-code-tip push, the given-up code branch delete and `ReturnToPreviousTransition` do not run in this layout.
  Teardown tags `archive/<slug>/<tip>`, pushes the tag, then deletes the branch locally and on origin, and the view ref too.
- **Replay.**
  Pull, origin reconcile and rejected-push recovery replay only records-only commits.
  A replayed commit's `Warp-SHA` is rewritten only when it names a replayed commit, to the fork point.
  When another local commit is unpushed and origin diverged, nothing is rewritten and the push or pull stops as owed.
- **Guarded prime push.**
  The prime's records push goes out only when every commit origin lacks is a records-only commit, a merge fabric wrote,
  or a commit such a merge brings in.
  Otherwise it is owed as a new `operator_unpushed` failed-push class, with a creel notice naming the operator's commits.
  A prime push that origin refuses as a protected branch records a `protected` class that is not owed;
  `statuscommit` completes the transition with its records unpushed and logs a warning.
  `lym fabric clone` records the prime's branch in `lym.primeBranch`;
  fabric refuses a records commit in the prime when it is off that branch or the key is absent.
- **Other packages.**
  `hubforge` builds a one-repository hub beside the two-repository one.
  `landingshed` reads the protected-branch match from fabric's push classification and names only `lym fabric merge --abort`
  as the way forward out of a merge stopped before the records step.
  `battencli` and `loomcli` route their commit-status wiring through the new seam.
  `internal/gitrepo` gains the merge and push support these paths use.
- **Docs.**
  `README.md` and `docs/getting-started.md` describe cloning a one-repository hub and its hazards:
  `--single-repo` on an existing two-repository hub's code repository starts a separate hub,
  switching, resetting or stashing in the prime holds or drops its records,
  and a prime whose branch refuses direct pushes keeps its records unpushed.
  `PATTERN-single-repo-hub` is new; `PATTERN-run-fsm`, `PATTERN-cwd-resolution`, `PATTERN-fabric-boundary`,
  `PATTERN-lym-files` and `PATTERN-pair-remote` carry the matching changes.

## Design decisions

- The layout is read from the tracked marker, never from a config key, so it reads right during clone and with siblings missing.
- Records strip by path at every merge, never by commit, so an agent's own `git add -A` that puts records into a code commit is tolerated.
- A commit message alone never admits a commit that changes code paths to replay or to the prime guard.
- Fabric rewrites only its own unpushed records-only commits.
- A squash landing's commit and a merge fabric wrote over hand-resolved conflicts are owed as `operator_unpushed` until they reach origin.

## Known limits

- The prime's records commits sit on the operator's code branch, so an operator who rebases it rewrites them too.
- A records write after a merge stopped before the records step, to a path the source changed, is lost on abort.
- While the prime's branch refuses direct pushes, its records stay unpushed and prime runs act past those transitions.

## To check by hand

- `add-anchor-does-not-move-forked-records` (`internal/fabricengine/addanchor.go`): adding an anchor does not move the forked records.
- `battencli-one-repo-seam-untested` (`internal/battencli/commitstatus.go`): the one-repository seam has no test of its own.
- `merge-over-parked-merge-pinned-for-mergein-only` (`internal/fabricengine/merge.go`):
  the refusal of a merge over a parked merge is pinned for the merge-in case only.
- `origin-reconcile-one-repo-code-stop-untested` (`internal/fabricengine/originreconcile.go`):
  the one-repository stop on a local code commit has no test.
- `readme-d2-clone-description-missing` (`README.md`): confirm the clone description now covers everything decision D2 requires.
- `replay-nonreplayed-warp-sha-untested` (`internal/fabricengine/replay.go`):
  a `Warp-SHA` naming a commit that is not replayed, which must stay, has no test.
- `run-fsm-background-protected-contradicts-d7` (`pattern/PATTERN-run-fsm.md`):
  confirm the background on the `protected` push class agrees with the `protected` exception in the transition rule.
- `Topology.Reconcile` still calls `retireBoltAnchorMarker` in a one-repository hub (`internal/fabricengine/anchormarker.go`),
  which can delete the board's marker.
- Two `TestReplay_OneRepository` rows were left out: `ReconcileReportsTheBranchDiverged` and `OriginAtAnotherAnchorIntegrates`.
- The pre-push origin reconcile blocked on a remote-refs lock in a one-repository hub,
  so the new `statuscommit` rows skip that check.
- The integration suite's `final` gate failed and was not attributed to this change:
  `internal/reedcli` is flaky and passed on rerun.
- A fix commit, `a823f323`, makes a landing test expect the `Fabric-Merge` trailer on the landing commit message;
  it cleared `internal/landingshed.TestFinalize_OverRealHub/resolves_a_conflict_and_squash_merges_into_the_parent`.
