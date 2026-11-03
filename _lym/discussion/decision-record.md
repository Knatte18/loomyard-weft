# Decision record — batten-remote-order

## Goal

A batten run never fails pass after pass because its task worktree is gone, and a slug's remote work stays in order: a teardown's archive tag never goes out ahead of the weft tip it archives, and no removal or deletion of a slug's former pair touches a later run's branches.

## Scope

In scope:

- The run-scope origin reconcile and the resume path of `lym batten run` when the task worktree is gone at a row other than Worktree-Create and Worktree-Teardown: re-adopt a live pair, or fail the run once with a stop naming the recovery.
- The way forwards that name `lym fabric remove <slug>` for a slug that has a batten run record, and the report of `lym fabric remove` itself on such a slug.
- The archive step of a pair's teardown and of the cleanup sweep: push order of the weft tip and the archive tag, the archive of a diverged origin tip, and the archive-tag push's retry on "missing necessary objects".
- The tackler stencil's rule for a verdict that rests on a commit or branch being absent from a remote.
- Every deletion of a slug's weft branch on origin: leased on the origin tip the deleter archived.
- The per-slug teardown lock in `internal/fabricengine/teardownlock.go`, which `fabricengine.Topology.Add`, the task-branch discard, the cleanup sweep and every push of a pair's weft branch also take.
- The remote-tracking ref and upstream config a former pair leaves under a branch name that `fabricengine.Topology.Add` forks anew or adopts.

Out of scope:

- Making remote work async, the subject of the board note `fabric-remote-async`.
- Finishing a paused run whose work already landed, the subject of the board note `landed-run-left-paused`.
- Serialising every remote operation of a slug, fetches and origin judges included, under one lock.

## Decisions

### D1 — A resume whose task worktree is gone judges origin, then re-adopts a live pair

The run scope of the origin reconcile gains an option that only `lym batten run`'s resume sets.
Under it, a run whose pair worktree is absent is no error: the slug still joins the pairs the reconcile classifies, its entry is judged from origin's Bolt and the batten run record, owner and ended included, no branch of the absent pair is judged, and the report marks the pair absent.
The parent's branch is not judged while the pair is absent; D1's second reconcile, run once the pair stands again, judges it with the pair's branches.
Every other caller of the run scope, `fabricengine.DropLocal` reached from `lym adopt` among them, leaves the option unset and keeps today's error from `originTargets` in `internal/fabricengine/originreconcile.go`.
A continue pass whose task worktree is absent is refused, which dobby reads as a full pass owed, so the next pass is a resume.

When `lym batten run` resumes a run whose status stands at a row other than Worktree-Create and Worktree-Teardown and whose task worktree is absent:

1. It runs that run-scope reconcile; an entry origin reads as ended, stopped or owned by another machine refuses as any resume does, and an unreachable origin refuses the pass with the re-run as its way forward.
2. It runs `git worktree prune` in the code and records repositories, which drops every stale registration, the slug's included, as `removeSibling` already prunes before a branch deletion.
3. It calls `fabricengine.Topology.Add` with no `Parent` and no `Anchor`, with `AdoptLocal` set, and with a new adopt-only option.
   Under the adopt-only option `Add` forks nothing: it returns a not-live sentinel instead of forking, right after its weft probe when the weft is not live, and after its warp probe when the task branch is neither local nor on origin.
   It also skips every check that guards only a fork it never makes: the source-dirt check, the read of the acting worktree's HEAD with its detached-or-unborn refusal, and the fork point's empty own marker refusal and anchor-move stop (`internal/fabricengine/add.go`).
   The adopted weft branch's origin record names the pair's parent, so no parent is resolved, and a parent pair removed since cannot refuse the re-adopt.
   When the records worktree directory still stands as a registered worktree of the records repository on the pair's weft branch, `Add` reuses it instead of refusing it: it adds no weft worktree, takes the anchor that worktree records, makes only the task worktree over the local or origin task branch, and wires the pair as its adopt path does; its rollback leaves that worktree in place, as it leaves an adopted branch.
   The `AdoptLocal` refusal of a records branch checked out in another records worktree does not count that reused worktree as another.
   A live weft is adopted at the anchor it records, so the pair of a beam project or of a subpath anchor comes back at its own anchor; `Add` refuses a non-empty `Anchor` over a live weft, and D1 passes none.
   `Add` judges liveness by its existing rule: the weft branch exists locally, or on origin with no `archive/<slug>/*` tag covering its tip.
4. After `Add` returns it spawns the pair push the way the Worktree-Create row does, then runs the run-scope reconcile again, which now judges the pair's branches, and resumes on its verdict.

D1's test of the task worktree reads a path at its location that does not resolve as a worktree as absent, unlike `taskWorktreePresent` in `internal/battencli/wire.go`, so a stray path reaches `Add`'s refusal of it instead of failing the pre-run every pass.

Bound: a pass may now re-create a task worktree with no operator step.
A pair whose task worktree alone is gone, its records worktree standing on its weft branch, is re-made around that records worktree.
It never re-creates a pair whose weft branch is gone locally and whose origin copy is absent or covered by an archive tag, which is a pair a removal tore down on purpose, nor one whose task branch is gone locally and on origin, nor one origin reads as owned by another machine, ended or stopped, nor one whose records worktree directory stands without being a registered worktree of the records repository on the pair's weft branch, which `Add` refuses.
A removal that left the local weft branch behind, which `removeSibling` tolerates once the weft worktree is gone, leaves a pair D1 re-adopts, since `Add` reads a local weft branch as live without reading the tags; D2's Land way forward rests on that.
`AdoptLocal` lets it take over the slug's own unmarked local warp branch, the branch the run itself made.
D1 ends the rule that only the `--adopt-local` flag sets `AdoptLocal` (`AddOptions.AdoptLocal` in `internal/fabricengine/add.go`, `internal/fabricengine/doc.go`, `internal/fabriccli/doc.go`), and replaces it with this one: the flag sets it, and otherwise only D1's resume does, under the adopt-only option, which forks no pair over a stale branch.

### D2 — A pair that cannot be re-adopted fails the run once, with a way forward by row

D1 ends in a failed stop only on a permanent answer from `Add`, a closed set of typed refusals:

- the not-live sentinel, for a weft that is not live or a task branch gone locally and on origin;
- `ErrRemoteLeftover`, which under the adopt-only option is only a local weft branch that diverges from origin's;
- the refusal of a records worktree directory that stands without being a registered worktree of the records repository on the pair's weft branch, which `Add` returns as a typed error instead of today's plain one;
- the `AdoptLocal` refusals of a task branch checked out in another worktree and of a records branch checked out in another records worktree (`refuseAdoptLocalCheckedOut` in `internal/fabricengine/addadopt.go`), which `Add` returns as typed errors;
- the refusal of a path that stands at the task worktree's location, which `Add` returns as a typed error.

Every other answer is transient and refuses the pass as today, so dobby retries and its failing-pass notice applies: a transport or probe failure, the teardown-lock wait of D7 running out, a failed `git worktree add`, and any refusal outside the closed set.
The pre-run tests the closed set with `errors.Is` and `errors.As`, so a transient answer needs no sentinel of its own.

On a permanent answer, for a `running` status, the pre-run writes a failed stop through `shedengine.WriteFailedStop`, commits the status and spawns the detached status push, as a batten status commit does elsewhere.
A Busy answer from the run lock refuses the pass as transient.
Its error names the cause, `Add`'s refusal text, and its way forward depends on the row:

- At a row before Land, on the not-live sentinel or `ErrRemoteLeftover`: `lym batten cancel <slug> --remove`, which removes the pair through `removePairAndBranch` in `internal/battencli/cancel.go` and archives and deletes the task branch, so it clears the run's record, its claim and the pair parts `Add` refused on.
- At Land, whose work has already reached the parent, on the not-live sentinel or `ErrRemoteLeftover`: the steps that clear the refusal, then `lym batten run <slug>`, never a cancel, which would release a landed entry.
  - On the not-live sentinel, the pre-run names the restore of each branch that is not live, and of no other.
    When the weft is not live, it names the tag that holds the run's records.
    Its candidates are the slug's `archive/<slug>/*` tags on origin and the records repository's local `archive/<slug>/*` and `set-aside/<slug>/*` tags.
    The pre-run fetches the commit of each candidate only origin holds by SHA into the records repository, writing no branch or tag, and keeps a candidate whose commit holds the child run's status at `done`, the state Land resumes over.
    Of those it takes the one whose commit has the latest committer date: no later run of the slug can have started while this run's record stands, so an earlier run's tag is older, and a diverged origin tip archived beside the run's own holds no `done` child status of a later state.
    It fills that tag into `git -C <records repo> fetch origin tag <tag>`, for a tag only origin holds, and `git -C <records repo> branch <weft branch> <tag>`.
    With no such tag, no copy of the run's records is left on origin or in the hub, as after the removal of a local-only pair, and the stop says so and names finishing the landing by hand, the case of the board note `landed-run-left-paused`.
    When the task branch is gone locally and on origin, it names `git -C <prime> branch <task branch> <parent branch>`: Land's own work is the prime's pull, deploy, board prune and ready notice (`NameLand` in `internal/battenrecipe/names.go`), and the task's work already sits on the parent.
    The restored branches are live by `Add`'s rule, so the re-run's D1 adopts them, the child's run records with them.
    It names running `lym batten run <slug>` right after the restore: a `lym fabric cleanup --apply` between the two reads a restored weft branch, which has no worktree, as an orphan and archives and deletes it, and the restore must then be run again from the tag, which survives.
  - On `ErrRemoteLeftover`: the pre-run names archiving origin's diverged tip and deleting it under a lease on that tip, with each command filled in: `git -C <records repo> fetch origin <tip>`, `git -C <records repo> tag archive/<slug>/<first 12 hex of tip> <tip>`, `git -C <records repo> push origin archive/<slug>/<first 12 hex of tip>`, `git -C <records repo> push --force-with-lease=refs/heads/<weft branch>:<tip> origin --delete <weft branch>`, then, only when the branch has an upstream configured, `git -C <records repo> branch --unset-upstream <weft branch>`.
    The local weft branch, which holds this machine's run records, is then live with nothing on origin, and its next push runs as a first push.
- At any row, on the other answers: the step that clears the obstacle, then `lym batten run <slug>`, which resumes the run from its record; no work is lost, so no row needs a cancel.
  - On a records worktree directory that is not the pair's: the pre-run names the directory; once anything in it worth keeping is saved, `git -C <records repo> worktree remove --force <dir>` when it is a registered worktree, else deleting the directory.
  - On a branch checked out in another worktree: the refusal's own step, `lym fabric checkout <other branch>` in the worktree holding the task branch, or switching the records worktree holding the records branch off it; the stop names `lym batten run <slug>` after that step in place of the refusal's re-run of `lym fabric add <slug> --adopt-local`.
  - On a path at the task worktree's location: the pre-run names the path; once anything in it worth keeping is saved, deleting it.

The re-run finds the status `failed` at Land, which `lym batten run` resumes from its persisted row as it resumes any failed status (`checkRunStatus` in `internal/battencli/arm.go`); D1 then re-adopts the pair, and the pass resumes Land on the second reconcile's verdict.

The pass posts one notice to the run's parent naming the slug and the way forward, through the notice path a batten hand-back uses, and exits non-zero.
A status other than `running`, the `failed` status of such a re-run included, keeps its state on a permanent answer, and the pass refuses with the same error and the way forward its answer calls for.

Bound: the pre-run gains one status write, `failed` with a way forward, only when the task worktree is absent and the answer is permanent.

### D3 — A way forward naming `lym fabric remove` on a batten run's slug also names the step that settles its record

Every stop or refusal that names `lym fabric remove <slug>` as a way forward for a slug whose batten run record stands at a row before Land names `lym batten cancel <slug> --remove` after it, as the step that clears the run's record and the task branch the next run's `Add` would refuse.
One whose record stands at Land names D2's Land way forward after it instead.

Finalize's stop when origin's parent already holds the task's work, in `internal/landingshed/finalize_owner.go`, is the exception: it names no cancel, and it keeps leaving the owner pause standing.
A cancel lifts every slug-scope pause, the owner pause among them (`liftSlugPauses` in `internal/battencli/cancelcmd.go`).
The record stays under the owner pause, which keeps a pass from driving it.
Another machine has landed its work, so settling it is the case of the out-of-scope board note `landed-run-left-paused`, and the stop's text for it stays as today.

`lym fabric remove`, after it removes the pair of a slug whose batten run record stands, reports a warning in its result and changes no batten record itself: before Land the warning names `lym batten cancel <slug> --remove`, and at Land it names D2's Land way forward.
A record at Worktree-Teardown or `done`, and a slug an owner pause covers, get no warning.

Rationale: `childReachedParent` in `internal/battencli/cancel.go` returns true with the child's status gone only when batten stands at Worktree-Teardown or the run is past its parent-side merge.
A cancel before Land therefore clears the record, and a cancel at Land with the merge not yet read would release a landed entry, the outcome D2 forbids.

### D4 — A teardown pushes the weft tip before the archive tag that names it

In `fabricengine.Topology.Remove`, under the pair's exclusive teardown lock and after the pending records are committed, the archive step:

1. Reads the weft branch's origin tip.
2. When origin lacks the local tip, pushes the weft branch alone, rebase-free, under the records repository's remote-refs lock; the code branch is not pushed, and a refused push writes no failed-push record or notice, under the `PATTERN-push-both-sides` change below.
3. Pushes the archive tag at the local tip, and also an archive tag at origin's tip when that tip, after step 2, is neither the local tip nor an ancestor of it.
   That origin tip is fetched by SHA before its tag is made, as `resolveArchiveTip` already fetches an origin-only tip, since the tag is made locally.
4. Retries an archive-tag push whose remote line holds "missing necessary objects", within the archive-tag push's existing delays.

Any failure of the branch push is logged and the tag pushes go on, as today; it never fails the removal.
The branch push passes the same pause gate the tag push already passes, so a covering pause holds `Remove` at the point it holds it today.
A pair with no weft origin, or a local-only pair, pushes no branch.

Rationale: a detached push of the pair that starts while the teardown lock is held exits without pushing, so the teardown is the last chance to publish the weft tip; archiving a diverged origin tip too leaves no tip on origin that `Add` would read as live.

Bound: `Remove` waits on one more network push, a departure from the direction of the board note `fabric-remote-async` that only the teardown itself waits on.

### D5 — The tackler checks the remote before judging a commit or branch absent from it

The tackler stencil, `contracts/stencils/shed/shed-template-tackler.md`, gains a rule: a verdict that rests on a commit or branch being absent from a remote first reads the remote tip with `git ls-remote <remote> <ref>`.
A branch is absent when `ls-remote` answers no line for its ref.
The commit the stop names is present when it equals the tip, or when it is an ancestor of the tip by `git merge-base --is-ancestor` where the tip is known locally; otherwise it is unsettled, and never read as absent.

### D6 — Every deletion of a slug's weft branch on origin is leased on the origin tip the deleter archived

The weft branch's deletion on origin, in a removal's weft teardown, in its origin-only path and in the cleanup sweep, uses the leased remote delete the task branch's discard already uses.
In a removal, the lease is origin's weft tip after D4's branch push: the local tip when that push landed, else the tip D4 step 1 read, and an archive tag of D4's covers that tip, at it or at a descendant.
In the origin-only path, the lease is the origin tip the archive step read and archived, never a second read of origin.

The cleanup sweep follows D4's archive rule: under the slug's teardown lock it reads the weft branch's origin tip, archives that tip as well, fetched by SHA, when it is neither the local tip nor an ancestor of it, and leases the remote delete on it.
Today the sweep archives only the local tip (`Cleanup` in `internal/fabricengine/cleanup.go`, `resolveArchiveTip` in `internal/fabricengine/archive.go`).

A delete the lease refuses keeps the branch on origin and is reported in the result as kept because origin moved during the removal, never as a failure of the removal.

Bound: a branch kept this way was pushed by another process between the read and the delete, which under D7 is no new pair of this slug on this machine; it is another machine's live branch, and `Add` adopting it is correct.

### D7 — A slug's pair is made, removed, discarded and swept under its teardown lock

- `fabricengine.Topology.Add` takes the slug's teardown lock exclusive from before its origin probes to its return, waiting as long as `Remove` waits; a wait that runs out refuses `Add` before any mutation, naming the slug's running push or removal and the re-run.
- `fabricengine.Topology.DiscardTaskBranch` takes it exclusive across its archive and remote delete, with the same wait.
- The cleanup sweep takes it with one try per slug before that slug's archive; a held lock skips the slug whole, its archive and its local and remote deletes, and reports it skipped.
- The callers of `Add` spawn the new pair's push after `Add` returns, as today, so the push child takes the lock shared once `Add` has released it.
- Every push of the pair's weft branch from this machine outside a removal, the synchronous `commit-records` push included, takes the slug's teardown lock shared and exits without pushing when a removal holds it exclusive, as the detached push does.

Rationale: no new pair's origin probes or first push interleave with the removal, discard or sweep of the slug's former pair.

Bound: `Add` and the discard wait up to the teardown wait; neither waits on another slug's work, and fetches and origin judges stay outside the lock.
D4 lengthens `Remove`'s exclusive hold by one branch push and its archive-tag pushes, each retrying on `archiveTagPushDelays`, about a minute of delays per tag, which can outrun the teardown wait.
A hand `lym fabric add` or a cancel's discard that meets such a hold is refused before any mutation, naming the running removal and the re-run; D1's re-adopting `Add` that meets it is transient under D2, so dobby retries it.
A `commit-records` commit made after D4 read the local tip is given up with the removal, as a detached push's commit is today: neither is pushed, and no archive tag covers it.

### D8 — `Add` drops a former pair's remote-tracking ref for a branch origin lacks

When `fabricengine.Topology.Add` forks a branch, weft or warp, whose origin copy its pre-flight probe found absent, or adopts a local branch whose origin copy is absent, it deletes that branch's remote-tracking ref `refs/remotes/origin/<branch>` in its repository and, when the branch has one, unsets its upstream config, `branch.<name>.remote` and `branch.<name>.merge`, in one step under the repository's remote-refs lock, once its pre-flight refusals have passed and before its worktrees are made.
An adopted local weft branch's origin copy is read by the weft probe.
An adopted local warp branch's is read by the warp probe, which `Add` skips for such a branch except under the adopt-only option, which runs it; there the probe's answer serves only this drop and never makes `Add` fetch the task branch from origin.

Rationale: the run-scope reconcile judges every branch that has a tracking ref and reads one origin lacks as gone (`originTargets` and `judgeBranch` in `internal/fabricengine/originreconcile.go`), so a tracking ref a former pair left behind makes a new or re-adopted pair's branch read gone until its first push lands; the 2026-11-03 failure under Constraints is that case, and D1's second reconcile and D2's Land restore would meet it too.
A branch whose upstream config outlives its tracking ref fails every push: `UpstreamSHA` in `internal/gitrepo/blobread.go` returns a resolve error instead of `ErrNoUpstream`, which `checkOrigin` in `internal/fabricengine/pushclassified.go` passes up instead of letting the push run.
With both gone the branch reads as never pushed, and its next push runs as a first push.

Bound: the deleted ref names a branch origin does not hold, so no judge loses an origin head; a rollback restores neither the ref nor the upstream config.
For D1's re-adopt, the first reconcile has judged the entry before `Add` runs, so the dropped ref hides no verdict the resume needs.
`Add`'s step-11 replace of an archived leftover already drops the ref through its own delete push.

## Constraints

- The cancel's remote deletions run synchronously before `lym batten cancel` returns, and the push it spawns is the prime's records push, which deletes no branch (`internal/battencli/cancel.go`, `internal/fabricengine/remove.go`).
- A detached push holds the slug's teardown lock shared and exits without pushing when a removal holds it exclusive, and a removal waits a bounded time for it (`internal/fabricengine/teardownlock.go`).
- `Add` reads an origin weft tip as covered, and so not live, only when an archive tag's target equals it or descends from it (`internal/fabricengine/remoteleftover.go`).
- The archive-tag push takes no remote-refs lock, as `PATTERN-remote-refs-lock` allows for a tag push; D4's branch push writes a tracking ref, so it runs under the lock.
- `PATTERN-push-both-sides` keeps per-transition status pushes detached, and D2's status push is the detached one.
- `PATTERN-owner-machine`'s Worktree-Teardown exception stays as it is; D1 keeps the reconcile first for every other resume.
- Branch names carry no run id, so successive runs of a slug share the code branch and the weft branch.
- The 2026-11-03 failure was run A's remote-tracking ref of `shed-sub-run-block-weft`, which outlived run A's removal and which run B's reconcile read gone at 08:18:44 before run B's own first weft push landed at 08:18:46; D8 covers it, and D6 and D7, which order only an overlapping removal, do not.
  The evidence is the hub's trace logs and the reconcile's read order, recorded in the support log's Review rounds.
- The 2026-11-02 "missing necessary objects" rejection comes from receive-pack's connectivity check, which rejects a ref update unless every object in the transitive closure of its tip exists on the remote (`git-receive-pack(1)` at git 2.53.0, `--skip-connectivity-check`).
  The pushing side leaves out only the objects reachable from the refs the remote advertised, so a tag push of a commit no advertised ref reaches carries that commit's objects and cannot fail the check.
  The rejection therefore means the remote advertised a ref reaching the tagged commit, the weft branch the pair's detached push was landing, to a tag push whose check read a store that did not yet hold its objects; the weft branch on origin at that commit minutes later fits.
  D4's branch push first makes the tag push follow a branch push this process saw land, and its retry covers a check that reads a store behind the advertisement.
  A remote that loses objects outright is not covered and needs no design here: the tag push then fails, and the removal keeps everything in place for its re-run.

## Auto-mode assumptions

Every pick is an auto-pick recorded in the support log's question ledger.

## Open risks

None.

<!-- lym:carry-over Discussion-Review -->
- Discussion-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/discussion/round-2-review.md; fixer report: _lym/reviews/discussion/round-2-fixer-report.md.
  - d1-task-worktree-alone-gone-failed (class design, severity MEDIUM, open in rounds 2)
  - d2-before-land-cancel-keeps-pair-parts (class design, severity MEDIUM, open in rounds 2)
  - d2-permanent-transient-split-by-exclusion (class design, severity MEDIUM, open in rounds 1, 2)
  - d6-bound-rests-on-lock-sync-push-skips (class design, severity MEDIUM, open in rounds 2)
  - d8-cause-asserted-not-shown (class design, severity MEDIUM, open in rounds 2)
  - d8-drop-leaves-upstream-config (class design, severity BLOCKING, open in rounds 2)
  - d8-local-warp-stale-tracking-ref (class design, severity MEDIUM, open in rounds 2)
<!-- /lym:carry-over Discussion-Review -->

<!-- lym:carry-over Plan-Review -->
- Plan-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/plan/round-2-review.md; fixer report: _lym/reviews/plan/round-2-fixer-report.md.
  - archive-test-reads-unexported-field (class consistency, severity MEDIUM, open in rounds 2)
  - cleanup-live-pair-reread-unreached (class consistency, severity MEDIUM, open in rounds 2)
  - d3-follow-up-sites-enumerated-by-hand (class design, severity MEDIUM, open in rounds 2)
<!-- /lym:carry-over Plan-Review -->

## Acceptance criteria

- D1: a resume at Run-Shed whose task worktree is gone and whose weft branch is on origin with no archive tag covering it re-creates the pair, spawns its push and resumes on the second reconcile's verdict; the status stays `running`.
- D1: the same resume with the slug's unmarked warp branch still local and its worktree registration stale prunes the registration and re-adopts the pair.
- D1: a resume of a beam project's run whose task worktree is gone re-adopts its live pair at the anchor its weft records.
- D1: a resume whose task worktree is gone while the prime has tracked dirt re-adopts the pair.
- D1: a resume whose weft is live but whose task branch is gone locally and on origin re-creates no pair and answers not live.
- D1: a resume whose task worktree is gone for an entry origin reads as owned by another machine re-creates no pair and refuses as an owned-elsewhere resume.
- D1: the run-scope reconcile with the resume's option, of a slug whose pair worktree is absent, reports the pair absent and the entry's verdict, and returns no error; without the option it returns today's error.
- D1: a continue pass whose task worktree is absent is refused.
- D1: a resume at Run-Shed whose task worktree alone was deleted, its records worktree, weft branch and task branch standing, re-creates the task worktree over the task branch, wires it to the standing records worktree and resumes on the second reconcile's verdict; the status stays `running`.
- D2: a resume at Run-Shed whose task worktree is gone and whose weft tip an archive tag covers commits a `failed` status whose error names `lym batten cancel <slug> --remove`, posts one notice, and dobby plans no further pass for it.
- D2: the same resume at Land, after a removal archived the weft tip, names the fetch and the `git branch` of the archive tag whose commit holds the child's `done` status, the `git branch` of the task branch at its parent, and `lym batten run <slug>`, not a cancel; running those steps and then the re-run resumes Land over the `failed` status with the child's run records present in the re-adopted records worktree.
- D2: a resume whose re-adoption fails on an unreachable origin, on D7's lock wait or on a failed `git worktree add` refuses the pass and leaves the status `running`.
- D3: Finalize's stop for origin's parent already holding the work names `lym fabric remove <slug>` and the standing owner pause, and does not name `lym batten cancel`.
- D3: `lym fabric remove` of a slug whose `running` batten run record stands before Land warns naming `lym batten cancel <slug> --remove`; one whose record stands at Land names D2's Land way forward and not `lym batten cancel`; one with no record, a `done` one, or one at Worktree-Teardown does not warn.
- D4: a removal whose weft tip origin lacks pushes the weft branch before the archive tag, and origin holds the tip when the tag push runs.
- D4: a removal whose origin weft tip diverges from the local tip fetches that tip, pushes an archive tag at each tip, and a later `Add` of the slug reads the pair as not live.
- D4: an archive-tag push refused once with "missing necessary objects" and accepted on retry archives the tip and the removal succeeds.
- D4: a failed weft branch push still pushes the tag and the removal succeeds.
- D5: the rendered tackler prompt carries the `git ls-remote` rule, under which a named commit that is an ancestor of the remote tip reads present.
- D6: a removal whose weft branch on origin moves between the remover's read and its delete keeps that branch and reports it kept; one at the read tip is deleted.
- D6: a removal whose weft tip origin lacked pushes it and then deletes the weft branch on origin under a lease on the pushed tip.
- D6: a cleanup sweep of an orphan whose origin weft tip diverges from the local one archives both tips and deletes the origin branch under a lease on its origin tip.
- D7: `Add` of a slug waits while `Remove`, the discard or the sweep of the slug holds the teardown lock exclusive, then makes the pair once the lock is released.
- D7: `Add` whose wait runs out refuses before any mutation, naming the running push or removal and the re-run.
- D7: a cleanup sweep that finds a slug's teardown lock held skips that slug whole, its archive and its deletes, and reports it.
- D7: a `commit-records` push of a pair whose removal holds the teardown lock exclusive pushes nothing.
- D8: an `Add` of a slug whose former pair's weft tracking ref outlived its removal leaves no tracking ref for the new weft branch, and a run-scope reconcile before the new pair's first push reads no branch of it gone.
- D8: a D1 re-adopt of a local weft branch that origin lacks, under a stale tracking ref, drops that ref, and its second reconcile reads no branch of the pair gone.
- D8: a D1 re-adopt of a local weft branch that origin lacks has its spawned pair push land the weft branch on origin.
- D8: a D1 re-adopt of a local task branch that origin lacks, under a stale tracking ref, drops that ref, and its second reconcile reads no branch of the pair gone.

## PATTERN changes

- `PATTERN-slug-remote-order` — adds: Touching a slug's pair or its origin branches: under fabricengine's per-slug teardown lock, shared for each pair push outside a removal, exclusive for Add, Remove, task-branch discard and cleanup sweep; a weft branch delete on origin is leased on the origin tip read.
- `PATTERN-push-both-sides` — changes: Pushing run records from Go: code branch goes along; a push refused after retry leaves a failed-push record and creel notice. A teardown's push of a weft branch it archives and deletes goes alone and only logs a refusal.

## Notes for the plan writer

- D2's status commit and push can follow the batten status commit in `internal/battencli/commitstatus.go`.
