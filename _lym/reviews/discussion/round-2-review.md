---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D8, with D1 step 4 and D2's ErrRemoteLeftover way forward
    summary: >-
      D8 drops the tracking ref of an adopted local weft branch origin lacks but leaves its upstream config,
      so UpstreamSHA errors and checkOrigin refuses every push of the re-adopted pair's weft, D1's pair push and every later status push included.
    fix: >-
      In D8, state that when Add drops the remote-tracking ref of an adopted local branch whose origin copy is absent,
      it also unsets that branch's upstream config (branch.<name>.remote and branch.<name>.merge) in the same step and under the same remote-refs lock,
      so the branch reads as never pushed and the next push runs as a first push (UpstreamSHA in internal/gitrepo/blobread.go returns ErrNoUpstream, which checkOrigin in internal/fabricengine/pushclassified.go lets through).
      Name the same unset in D2's ErrRemoteLeftover way forward, after the leased delete: git -C <records repo> branch --unset-upstream <weft branch>.
      Add an acceptance criterion under D8 that a D1 re-adopt of a local weft branch origin lacks has its spawned pair push land the weft branch on origin.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D2, the paragraph opening Every other answer is transient, and D1 step 3
    summary: >-
      D2 classes as transient every Add refusal outside its closed set, but several never clear on their own,
      so the pass fails every dobby pass with the status running, which is the failure the Goal forbids.
    fix: >-
      In D1 step 3, state that under the adopt-only option Add also skips the fork-point checks that guard only a fork it never makes:
      the acting worktree's HEAD read and its detached-or-unborn refusal, the empty own marker refusal of the fork point's weft and the fork point's anchor-move stop (all in internal/fabricengine/add.go),
      and replace the sentence With both branches adopted, no parent is resolved with that statement.
      In D2, add to the closed set of permanent answers the adopt-local refusal of a task branch checked out in another worktree (refuseAdoptLocalCheckedOut) and the refusal of a stray path at the task worktree's location, each as a typed error of Add,
      remove the fork point's anchor-move stop from the transient list,
      and state that D1's absent-worktree test treats a path at the task worktree's location that is not a resolvable worktree as absent, so the stray path reaches Add's refusal instead of erroring the pre-run every pass.
      Give each new permanent answer its way forward per row, as D2 does for the others.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D6 Bound, D7, and the PATTERN-slug-remote-order text under PATTERN changes
    summary: >-
      D6's bound says a branch the lease keeps is another machine's, but only the detached push takes the teardown lock shared;
      the synchronous commit-records push of the same pair takes none, so it can land a tip between D4's read and the leased delete that no archive tag covers and a later Add re-adopts.
    fix: >-
      In D7, add a bullet that every push of a pair's weft branch from this machine, the synchronous commit-records push included, takes the slug's teardown lock shared and exits without pushing when a removal holds it exclusive, as the detached push does.
      Change the PATTERN-slug-remote-order text in PATTERN changes, and the same text in the declared change, from shared for a detached push to shared for every push of the pair.
      Keep D6's bound sentence, and add an acceptance criterion under D7 that a commit-records push during a removal's exclusive hold pushes nothing.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md Constraints, the 2026-11-03 failure bullet, and D8 Rationale
    summary: >-
      The record asserts the tracking ref run B's reconcile judged was run A's, outliving run A's removal,
      but the removal's weft delete is a git push --delete, which drops that tracking ref on success, and the record cites no evidence such as the ref's reflog that ties the ref to run A.
    fix: >-
      Settle the link now from this hub's records repository and trace logs: read the reflog of refs/remotes/origin/shed-sub-run-block-weft in the records repository around 08:14-08:19 on 2026-11-03,
      and whether run A's removal ran the weft delete on origin (the removal's mutation record in the trace logs).
      Record under Constraints which process wrote the ref the reconcile judged at 08:18:44, with the reflog lines or log entries it rests on.
      If the ref was not run A's, rewrite D8's Rationale and the Constraints bullet to the cause the evidence shows, and change D8 so it covers that cause.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D8, first paragraph
    summary: >-
      D8 drops a stale tracking ref only for a forked branch or an adopted local weft branch, but D1 adopts a local warp branch under AdoptLocal without a warp probe,
      so a task-branch tracking ref of the kind D8's own premise says outlives a removal still makes D1's second reconcile read the task branch gone, D2's Land restore included.
    fix: >-
      In D8, extend the drop to an adopted local warp branch: under the adopt-only option Add probes origin for an adopted local warp branch too,
      and when origin lacks it deletes refs/remotes/origin/<task branch> in the code repository under its remote-refs lock (and, per F1, unsets its upstream config).
      Add an acceptance criterion under D8 that a D1 re-adopt of a local task branch origin lacks, under a stale tracking ref, drops that ref and its second reconcile reads no branch of the pair gone.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D2, the bullet At a row before Land, and the first D2 acceptance criterion
    summary: >-
      The way forward before Land is lym batten cancel without --remove, which keeps the pair parts that made Add refuse,
      so the next run of the slug fails at Worktree-Create on ErrBranchExists, the standing records worktree or ErrRemoteLeftover.
    fix: >-
      In D2's At a row before Land bullet, name lym batten cancel <slug> --remove, which removes the pair through removePairAndBranch and archives and deletes the task branch,
      and say it clears the run's record, claim and the pair parts Add refused on.
      Change the first D2 acceptance criterion and D3's two mentions of lym batten cancel <slug> before Land to the same --remove form.
  - id: F7
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D2, the records-worktree permanent answer, and D1 Bound
    summary: >-
      A run whose task worktree alone is gone while its records worktree and both branches stand fails permanently, and before Land its way forward discards the run by a cancel,
      though the board entry expects the pair recreated or adopted from its branches when they still exist.
    fix: >-
      In D1, add that when the records worktree directory still stands as a registered worktree of the records repository on the pair's weft branch, the resume re-creates only the task worktree over the local or origin task branch and re-wires the pair through Add's adopt path, instead of failing;
      or, if the writer keeps the refusal, state in D2 why such a pair cannot be re-adopted and name for rows before Land a way forward that keeps the run, such as lym fabric remove <slug> then the restore D2 names at Land, instead of a cancel.
      Add an acceptance criterion under D1 or D2 for a resume at Run-Shed whose task worktree alone was deleted.
  - id: F8
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D2, Land, the not-live sentinel bullet
    summary: >-
      The newest of the slug's archive tags has no defined order, since archive tags are lightweight and named by SHA,
      and successive or adopted runs of a slug can each leave a tag whose commit holds the child at done.
    fix: >-
      In the not-live bullet, define the pick: among the slug's archive/<slug>/* and set-aside/<slug>/* tags whose commit holds this run's child status at done, filtered by the run id the batten status records,
      take the one whose tagged commit has the latest committer date; and state that the pre-run fetches a candidate tag's commit by SHA into the records repository to read it, writing no ref.
  - id: F9
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D2, Land, the not-live sentinel bullet, and D6 cleanup sweep paragraph
    summary: >-
      The Land restore leaves a local weft branch with no worktree, which the cleanup sweep reads as an orphan and archives and deletes, undoing the restore before the re-run.
    fix: >-
      In D2's not-live bullet, add that the restore is undone by a lym fabric cleanup --apply run between the restore and the re-run, so the way forward names running lym batten run <slug> right after the restore;
      or state in D6's sweep paragraph that the sweep skips a weft branch whose slug has a batten run record standing.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D1, second paragraph
    summary: >-
      D1 says the absent pair's entry is judged from its origin record and the Bolt, but the origin record lives in the absent records worktree, so ReadOriginFor finds none and the parent branch drops out of the reconcile's scope.
    fix: >-
      In D1's second paragraph, replace judged from its origin record and the Bolt with judged from origin's Bolt and the batten run record,
      and state whether the parent's branch is judged when the pair is absent; if it must be, name where the run-scope reconcile reads the parent from, such as the batten status or the weft branch's committed origin record read by git show.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D3, second paragraph
    summary: >-
      D3 says a cancel would refuse as owned elsewhere in the adopter-first case, but that stop is reached only after Finalize's owner checks let this machine land, so the local board shows this machine as owner.
    fix: >-
      In D3's second paragraph, delete the clause and refuses as owned elsewhere when the local board shows another owner, which is this stop's adopter-first case (cancelOwnedElsewhere in internal/battencli/cancel.go),
      keeping the owner-pause lift as the reason the stop names no cancel.
  - id: F12
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md Constraints, the 2026-11-03 failure bullet
    summary: >-
      The Constraints bullet carries a seven-line timeline of run A and run B with timestamps, deliberation narrative the plan writer does not act on beyond its conclusion.
    fix: >-
      Keep in Constraints one sentence stating the settled cause and its evidence source (per F4's outcome) and the fact D8 rests on;
      move the timestamped timeline of run A and run B to the support log's Interview or Review rounds section.
  - id: F13
    severity: NIT
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D6, second paragraph
    summary: >-
      D6 says D4 archives exactly the lease tip, but when D4's push fails and origin's tip is an ancestor of the local tip only the local tip is tagged, covering the lease tip by a descendant.
    fix: >-
      In D6's second paragraph, replace and D4 archives exactly that tip with and an archive tag of D4's covers that tip, at it or at a descendant.
  - id: F14
    severity: NIT
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D1 step 2
    summary: >-
      D1 step 2 says it prunes the slug's stale worktree registrations, but git worktree prune is repo-wide and cannot be scoped to one slug.
    fix: >-
      In D1 step 2, replace prunes the slug's stale worktree registrations with runs git worktree prune in the code and records repositories, which drops every stale registration, the slug's included.
  - id: F15
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md Open risks, Constraints fifth bullet, Notes for the plan writer
    summary: >-
      Open risks narrates history instead of stating none, and the push-both-sides carve-out and the typed records-worktree refusal are each stated twice or three times across D2, D4, Constraints and Notes.
    fix: >-
      Replace the Open risks line with None.
      Delete the Constraints bullet's second clause from D4's push of the weft branch alone onward, and delete the first and third bullets of Notes for the plan writer, since D4 step 2 and D2's closed set already state them.
---

### [BLOCKING] D8's tracking-ref drop leaves the upstream config, so every push of the re-adopted weft fails

**Location:** `_lym/discussion/decision-record.md` D8, with D1 step 4 and D2's `ErrRemoteLeftover` way forward

**Issue:** D8 deletes `refs/remotes/origin/<weft>` for an adopted local weft branch that origin lacks.
That branch was pushed by the former pair, so its `branch.<weft>.remote` and `branch.<weft>.merge` config stands.
`UpstreamSHA` (`internal/gitrepo/blobread.go:264-291`) finds the config, fails to resolve the missing tracking ref and returns a wrapped error, not `ErrNoUpstream`.
`checkOrigin` (`internal/fabricengine/pushclassified.go:391-397`) returns `(false, upstreamErr)` for any error but `ErrNoUpstream`.
So D1 step 4's pair push and every later status push of the re-adopted pair refuse before they run, and the run's records never reach origin.
This hits D1's own named case, a removal that left the local weft branch, which D2's Land way forward rests on.
It also hits D8's second acceptance criterion and D2's `ErrRemoteLeftover` way forward, whose hand `git push --delete` drops the tracking ref the same way.
The record never mentions the upstream config.

### [MEDIUM] D2's transient bucket holds refusals that never clear, so the pass fails forever

**Location:** `_lym/discussion/decision-record.md` D2, the paragraph opening "Every other answer is transient", and D1 step 3

**Issue:** D1 calls `Add` with an empty `Parent`, so `Add` still runs the fork-point checks.
It reads the prime's HEAD and refuses a detached or unborn one (`add.go:351-361`).
It also refuses an empty own marker on the prime's weft (`add.go:255-258`) and a pending anchor move there (`add.go:259-266`), which D2 names transient by name.
None of these checks guards anything under adopt-only, which forks nothing.
Two more refusals stay outside the closed set too.
One is `refuseAdoptLocalCheckedOut`, which `AdoptLocal` reaches when the task branch is checked out in another worktree (`add.go:341-345`).
The other is a stray path at the task worktree's location.
`taskWorktreePresent` (`internal/battencli/wire.go:268-280`) returns that path as an error, not as absent, so D1 never runs.
Each case leaves the status `running`, and dobby retries every pass.
That is the "fails pass after pass" the Goal rules out.
The record's claim "With both branches adopted, no parent is resolved" also overstates: the acting worktree's HEAD is still read.
The existing-paths and interactions lenses raised the same defect.

### [MEDIUM] D6's bound rests on a lock the synchronous records push never takes

**Location:** `_lym/discussion/decision-record.md` D6 Bound, D7 and the `PATTERN-slug-remote-order` text

**Issue:** Only `DetachedPush` takes the teardown lock shared (`internal/fabricengine/teardownlock.go:75-87`).
The synchronous `commit-records` push of the same pair, which `PATTERN-push-both-sides` allows, takes none.
Take an agent in the pair that commits and pushes tip T2 after D4 archived local tip L and before the leased delete.
The lease refuses, the branch stays on origin at T2, and the tag at L is an ancestor of T2, not a descendant.
So T2 is uncovered, and `probeWeftLeftover` (`remoteleftover.go:105-126`) reads it live, so a later `Add` re-adopts the removed run's weft.
That contradicts D6's bound that a kept branch "is another machine's live branch".
The existing-paths lens raised the same defect.

### [MEDIUM] The 2026-11-03 cause D8 rests on is asserted, not shown

**Location:** `_lym/discussion/decision-record.md` Constraints, the 2026-11-03 bullet, and D8 Rationale

**Issue:** The cancel's removal ran with `Remote: true` (`internal/battencli/cancel.go:636`).
Its weft delete is `git push origin --delete <branch>` (`internal/gitrepo/push.go:192-195`, reached from `removeWeftWorktree`).
A successful delete-push also deletes `refs/remotes/origin/<branch>` when the remote has a fetch refspec.
A refspec must exist, since `originTargets` judges only branches with tracking refs.
Either the delete ran and dropped run A's ref, or it did not run and origin still held the branch, which contradicts the "gone" verdict.
The record cites trace logs for the timeline but nothing, such as the ref's reflog, that the ref judged at 08:18:44 was run A's.
D8's claim to cover the trigger is therefore a fact the design rests on that is left unsettled, which the rubric's deferred-research bullet flags.

### [MEDIUM] D8 leaves an adopted local task branch's stale tracking ref in place

**Location:** `_lym/discussion/decision-record.md` D8, first paragraph

**Issue:** `Add` skips the warp probe for an adopted local warp branch (`add.go:388-394`).
D8 covers only a forked branch or an adopted local weft branch.
D1 adopts the local task branch under `AdoptLocal`, and D2's Land restore recreates it locally with `git branch <task branch> <parent branch>`.
Take a stale `refs/remotes/origin/<task branch>` of the kind D8's own premise says outlives a removal.
`originTargets` keeps that branch (`originreconcile.go:673-681`), and `judgeBranch` reads it gone (`originreconcile.go:539-541`).
D1's second reconcile then stops the resume before the spawned push lands, and D2's Land acceptance criterion fails.
The interactions lens raised the same defect at LOW, but it breaks D2's main path, so it is MEDIUM here.

### [MEDIUM] D2's way forward before Land keeps the parts the next run's Add refuses

**Location:** `_lym/discussion/decision-record.md` D2, "At a row before Land", and the first D2 acceptance criterion

**Issue:** A cancel without `--remove` keeps the pair (`internal/battencli/cancel.go:683`, "its pair was kept").
Only `--remove` reaches `removePairAndBranch` (`cancel.go:635-659`).
The next run's Worktree-Create calls an ordinary `Add` with no `AdoptLocal`.
That `Add` refuses a kept unmarked warp branch with `ErrBranchExists` (`add.go:291-293`), a standing records worktree (`add.go:328-331`) and a diverged local weft with `ErrRemoteLeftover` (`remoteleftover.go:154`).
The way forward therefore moves the endless failure from the resume to the next run's first row.

### [MEDIUM] A run whose task worktree alone is gone is failed, and before Land discarded

**Location:** `_lym/discussion/decision-record.md` D2, records-worktree permanent answer, and D1 Bound

**Issue:** The board entry expects batten to recreate or adopt the pair "from its branches when they still exist".
Take a task worktree deleted by hand while the records worktree, the local weft branch and the task branch all stand.
That is the most direct form of bug 1's "no pair worktree" refusal.
D1 sends it to `Add`, which refuses the standing records worktree (`add.go:328-331`).
D2 makes that permanent, and before Land names a cancel, which throws the run away although every part needed to rebuild the task worktree is present.
D1's Bound lists this pair as never re-created, and neither D1, D2 nor the support log's rejected alternatives says why it must not be.

### [LOW] "The newest" archive tag has no defined order

**Location:** `_lym/discussion/decision-record.md` D2, Land, not-live bullet

**Issue:** Archive tags are lightweight and named by SHA (`internal/fabricengine/archive.go:111-122`), so they carry no date.
Branch names carry no run id, and D4 now makes two tags per removal, so several tags whose commit holds a child at `done` can coexist.
One can come from an earlier landed run of the slug, and another from the other machine of an adopted run.
The record does not say how the pre-run reads a tag's commit either: `ls-remote` gives only the SHA.
The bound-breaker lens raised the same defect.

### [LOW] The cleanup sweep can undo D2's Land restore before the re-run

**Location:** `_lym/discussion/decision-record.md` D2, not-live bullet, and D6's sweep paragraph

**Issue:** The restore makes a local weft branch with no worktree.
`Cleanup` (`internal/fabricengine/cleanup.go:199-261`) treats any weft branch whose warp branch has no worktree as an orphan, and under `--apply` archives and deletes it.
The tag survives, so the loss is recoverable, but the record names no such window.

### [LOW] D1 claims the absent pair's entry is judged from its origin record

**Location:** `_lym/discussion/decision-record.md` D1, second paragraph

**Issue:** The origin record is a file in the records worktree, which is absent here, so `ReadOriginFor` finds none.
`originTargets` then drops the parent branch from the run scope (`originreconcile.go:627-637`).
The sentence names a source the reconcile cannot read in this case.

### [LOW] D3's owned-elsewhere reason does not hold for the adopter-first stop

**Location:** `_lym/discussion/decision-record.md` D3, second paragraph

**Issue:** `adopterFirstRepair` words a note on a failed parent push after `ownerChecks` let this machine land (`internal/landingshed/finalize_owner.go:30-75`, `127-161`).
At that point the board shows this machine as owner, so `cancelOwnedElsewhere` would not refuse.
The decision stands on the owner-pause lift alone (`liftSlugPauses`, `internal/battencli/cancelcmd.go:475-496`).

### [LOW] The 2026-11-03 timeline is narrative in Constraints

**Location:** `_lym/discussion/decision-record.md` Constraints, the 2026-11-03 bullet

**Issue:** Seven timestamped lines trace run A and run B.
The plan writer acts only on the settled cause and the fact D8 rests on.
That fact stays in Constraints and the timeline moves to the support log, which loses nothing the plan writer needs.

### [NIT] D6 overstates that D4 archives exactly the lease tip

**Location:** `_lym/discussion/decision-record.md` D6, second paragraph

**Issue:** After a failed D4 push, an origin tip that is an ancestor of the local tip is covered by the local tip's tag, not tagged itself.
The delete is still safe under the covered rule, so only the wording is wrong.

### [NIT] D1 step 2's prune cannot be scoped to the slug

**Location:** `_lym/discussion/decision-record.md` D1 step 2

**Issue:** `git worktree prune`, the command `removeSibling` uses (`internal/fabricengine/remove.go:320`, `516`), is repo-wide.

### [NIT] Open risks narrates history and the record repeats itself

**Location:** `_lym/discussion/decision-record.md` Open risks, Constraints fifth bullet, Notes for the plan writer

**Issue:** Open risks tells which facts were once deferred instead of stating none.
D4 step 2, the Constraints bullet and the first note each restate the `PATTERN-push-both-sides` carve-out.
D2's closed set and the third note both say the records-worktree refusal becomes a typed error.

## Focus verdicts

- D1's adopt-only `Add`: the non-empty `Anchor` refusal (`add.go:397-401`) cannot fire, since D1 passes no anchor.
  `ErrSourceDirty` (`add.go:239-246`) cannot fire under the adopt-only skip.
  The call passes no `Parent`, no `Anchor`, `AdoptLocal` and the adopt-only option, but the fork-point checks it still meets are F2.
- D2 at Land, the not-live restore: it restores the child's records once F1, F5, F8 and F9 are settled.
  The records-worktree path restores them through `lym fabric remove`, which finishes a pair whose task worktree is gone (`remove.go:150`), and the not-live restore.
  The `ErrRemoteLeftover` path restores them only once F1 is fixed.
  `lym batten run` resumes a `failed` status from its persisted row (`checkRunStatus`, `internal/battencli/arm.go:575-595`).
- D4 against the declared `PATTERN-push-both-sides` text: the push goes alone and only logs a refusal, so it complies.
  After D4's own push, D6's lease names the local tip, and when the push failed it names the tip D4 read (F13 on the wording).
  `Cleanup` today archives only the local tip (`resolveArchiveTip`) and deletes on origin unleased, so it can delete an origin tip it never archived; D6 closes that.
- D3: a cancel at Land with the pair gone reads the child as not reached.
  `childReachedParent` needs the child status or past-merge (`cancel.go:691-708`), so a cancel would release a landed entry, as D3 states.
  A cancel lifts every slug-scope pause, the owner pause included (`liftSlugPauses`).
  The Finalize stop's exception holds on that ground (F11 on its second reason).

## Rejected

- D3's warning in `lym fabric remove` against `PATTERN-fabric-isolation`: batten is not among the modules that entry closes off, and fabric already reads batten status for its anchor refusal (`internal/fabriccli/doc.go:97-99`), so it is no violation.
