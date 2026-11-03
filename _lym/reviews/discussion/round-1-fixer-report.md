# Fixer report — discussion, round 1

Every one of the 16 findings is fixed in `_lym/discussion/decision-record.md`, and the support log records the round.
Two are fixed differently from the review's proposal (F5, F11), and the self-check changed the F1, F2, F7 and F10 fixes further.

## Changes

- **F1** (fixed): D1 step 3 passes no `Anchor`, and under the adopt-only option `Add` skips the source-dirt check, citing `add.go`'s anchor-over-live-weft and `ErrSourceDirty` refusals.
  New D1 criteria cover a beam project's re-adopt at its recorded anchor and a re-adopt with a dirty prime.
  After the self-check, D1 also passes no `Parent`, since both branches are adopted and no parent is resolved.
- **F2** (fixed): D2's Land way forward now names, for each permanent refusal, the step that clears it:
  - not-live: restore the weft branch from the archive tag that holds the child's `done` status, and the task branch at its parent when it is gone;
  - a records worktree that still stands: `lym fabric remove` first;
  - `ErrRemoteLeftover`: archive origin's diverged tip, then delete it under a lease.

  D2 states that `lym batten run` resumes the `failed` status at Land (`checkRunStatus`).
  The Land criterion now requires that the steps resume Land with the child's records present.
- **F3** (fixed): this takes the declared-change branch of the proposal.
  A `PATTERN-push-both-sides — changes:` bullet carries the full new entry text with the teardown carve-out, within the 280-character cap.
  The Constraints bullet and the plan-writer note now cite it.
- **F4** (fixed): D1's first paragraph makes the absent-pair tolerance a run-scope option that only batten's resume sets.
  Under it the entry is classified from its origin record and the Bolt.
  Every other run-scope caller keeps today's error, `DropLocal` included.
  The reconcile criterion now covers both the option set and the option unset.
- **F5** (fixed differently): D2 lists Add's refusal classes.
  The permanent side is a closed set of typed refusals: the not-live sentinel, a typed records-worktree refusal, and `ErrRemoteLeftover`.
  Every other answer is transient, the listed transport, lock-wait, worktree-add and anchor-move cases included.
  I did not add the proposed sentinel for probe and transport failures. With the permanent set closed and typed, the transient side is everything else, and a new sentinel would only duplicate that test. Under the proposal, an untyped refusal would still land on the permanent side, which is the defect the finding names.
  A criterion now covers a failed `git worktree add` leaving the status `running`.
- **F6** (fixed): D3's rationale now reads `childReachedParent` correctly: with the child's status gone, it returns true only at Worktree-Teardown or past the parent-side merge.
  The appended cancel and `lym fabric remove`'s warning are limited to a record before Land, and at Land they name D2's Land way forward.
  A new criterion checks that a record at Land does not name a cancel.
- **F7** (fixed): this takes the first branch of the proposal. Finalize's adopter-first stop names no cancel and keeps the owner pause standing.
  The reasons are cited: `liftSlugPauses` lifts that pause, and `cancelOwnedElsewhere` refuses.
  The criterion is replaced. After the self-check, the record's fate is named as the case of the out-of-scope note `landed-run-left-paused`.
- **F8** (fixed): D6's removal lease is origin's tip after D4's branch push, and D4 archives that tip.
  A new D6 criterion covers it.
- **F9** (fixed): D6 extends D4's archive rule to the cleanup sweep: it reads the origin tip under the lock, archives a diverged tip fetched by SHA, and leases the delete on it.
  A sweep criterion is added.
- **F10** (fixed): I traced both risks and dropped both Open risks lines.
  The section itself is required by the discussion spec, so after the gate's `discussion-section-missing` finding it stays and now states that there are no open risks, pointing at Constraints.
  - 2026-11-03: the hub's trace logs, the weft history and the archive tags pin the trigger. Run B's Seed-Child reconcile read `shed-sub-run-block-weft` gone at 08:18:44, before its first pair push landed the branch at 08:18:46, through a tracking ref that run A's removal left behind. This is recorded under Constraints with that evidence. The new D8 drops that stale ref in `Add`. D6 and D7 do not cover the case, since the removal had finished before `Add` began.
  - "missing necessary objects": I pinned this against the connectivity check in `git-receive-pack(1)` at git 2.53.0, and Constraints states which cause D4 covers.
- **F11** (fixed differently): I took the review's alternative, rewording the bound, and not the proposed not-live rule for a local branch an archive tag covers.
  D1's bound now names a surviving local weft branch as a pair D1 re-adopts.
  I did not make such a branch answer not-live, because D2's Land restore depends on a local weft branch reading live.
- **F12** (fixed): D4 step 3 fetches the diverged origin tip by SHA before tagging it, as `resolveArchiveTip` does.
- **F13** (fixed): D5 reads the remote tip with `ls-remote`. A commit is present when it equals the tip or is an ancestor of it by `merge-base --is-ancestor`, and is otherwise unsettled, never absent.
  The D5 criterion covers the ancestor case.
- **F14** (fixed): D7's Bound states the hold D4 adds: one branch push and the tag pushes, about a minute of `archiveTagPushDelays` per tag.
  A hand `Add` or a discard that is refused by the hold names the re-run.
  D1's `Add` that is refused this way is transient under D2.
- **F15** (fixed): the first D7 criterion is replaced: `Add` waits while `Remove`, the discard or the sweep holds the lock, then makes the pair.
- **F16** (fixed): D1's Bound states that it ends the rule that only the `--adopt-local` flag sets `AdoptLocal`, naming the three places that state the old rule, and states the rule that replaces it.

## Self-check

- fix-consistency, MEDIUM, D7's sweep lock disagreed with D6's archive under the lock: fixed as proposed. D7 now takes the lock before the slug's archive and skips the whole slug, and its criterion was updated to match.
- fix-consistency, LOW, ledger Q5 was stale: fixed as proposed.
- fix-proposal, MEDIUM, the newest archive tag can be a diverged tip and not the run's: fixed with own change. I did not use the proposed local-tip tag, because the local tip is gone after the removal and cannot be read. The tag is chosen by its commit holding the child's `done` status.
- fix-proposal, MEDIUM, D8 missed a re-adopted local weft branch that origin lacks: fixed as proposed. D8 now also drops the ref when it adopts such a branch, and a criterion is added.
- fix-proposal, MEDIUM, an unforkable `Parent` causes an endless transient loop, and "forks nothing" was false for the warp side: fixed with own change. D1 passes no `Parent`, and the adopt-only option answers not-live when the task branch is gone on both sides. That keeps "forks nothing" true. A criterion is added.
- fix-proposal, MEDIUM, the Finalize exception named no way forward for the record: fixed with own change. The record's fate is named as the case of the out-of-scope note `landed-run-left-paused`, and the stop text stays as today. A new clearing step falls outside this task's scope.
- fix-proposal, LOW, D5 had no outcome for an absent branch: fixed as proposed. A branch is absent when `ls-remote` answers no line for its ref.
- fix-way-forward, MEDIUM, the Land restore forked a fresh warp branch: fixed with own change. D1 refuses not-live when the task branch is gone on both sides, and D2's Land way forward restores the task branch at its parent. That holds because Land's work is prime-side (`NameLand`).
- fix-way-forward, MEDIUM, the Finalize record had no way forward: merged with the fix-proposal finding on the same point.
- fix-way-forward, LOW, the "any clone" restore could not be filled in: fixed with own change. The pre-run now also searches the local `archive/` and `set-aside/` tags. With none found, the stop says the records are gone and names the `landed-run-left-paused` case.
- fix-claims, MEDIUM, "archives before every removal" was false (`RemoveLocal`, local-only pairs, no origin): fixed together with the previous item; the claim was removed.
- fix-claims, MEDIUM, the origin-only path read origin twice: fixed as proposed. The lease is now the tip the archive step read.
- fix-claims, LOW, the newest-tag claim: merged with the fix-proposal tag finding.
- fix-claims, LOW, the D7 Bound sentence also covered Worktree-Create's `Add`: fixed as proposed. The sentence now names only D1's re-adopting `Add`.
- fix-claims, LOW, the "19 s" figure was not supported by the evidence: fixed as proposed. The sentence now says the cancel committed at 08:18:20, before run B's pass began at 08:18:31.

Every fork returned a report.

## Disputed

None.

## Deferred

None.
