# Decision record — batten-remote-order

## Goal

A batten run never fails pass after pass because its task worktree is gone, and a slug's remote work stays in order: a teardown's archive tag never goes out ahead of the weft tip it archives, and no removal or deletion of a slug's former pair touches a later run's branches.

## Scope

In scope:

- The run-scope origin reconcile and the resume path of `lym batten run` when the task worktree is gone at a row other than Worktree-Create and Worktree-Teardown: re-adopt a live pair, or fail the run once with a stop naming the recovery.
- The way forwards that name `lym fabric remove <slug>` for a slug that has a batten run record, and the report of `lym fabric remove` itself on such a slug.
- The archive step of a pair's teardown: push order of the weft tip and the archive tag, and the archive-tag push's retry on "missing necessary objects".
- The tackler stencil's rule for a verdict that rests on a commit or branch being absent from a remote.
- Every deletion of a slug's weft branch on origin: leased on the origin tip the deleter read.
- The per-slug teardown lock in `internal/fabricengine/teardownlock.go`, which `fabricengine.Topology.Add`, the task-branch discard and the cleanup sweep also take.

Out of scope:

- Making remote work async, the subject of the board note `fabric-remote-async`.
- Finishing a paused run whose work already landed, the subject of the board note `landed-run-left-paused`.
- Serialising every remote operation of a slug, fetches and origin judges included, under one lock.

## Decisions

### D1 — A resume whose task worktree is gone judges origin, then re-adopts a live pair

The run-scope origin reconcile no longer errors when the run's pair worktree is absent.
It judges the run's entry against origin as it does today, owner and ended included, judges no branch of the absent pair, and reports the pair as absent.
A continue pass whose task worktree is absent is refused, which dobby reads as a full pass owed, so the next pass is a resume.

When `lym batten run` resumes a run whose status stands at a row other than Worktree-Create and Worktree-Teardown and whose task worktree is absent:

1. It runs that run-scope reconcile; an entry origin reads as ended, stopped or owned by another machine refuses as any resume does, and an unreachable origin refuses the pass with the re-run as its way forward.
2. It prunes the slug's stale worktree registrations in the code and records repositories, as `removeSibling` already prunes before a branch deletion.
3. It calls `fabricengine.Topology.Add` with the run's parent and anchor, with `AdoptLocal` set and with a new adopt-only option under which `Add` refuses with a not-live sentinel instead of forking a new pair.
   `Add` judges liveness by its existing rule: the weft branch exists locally, or on origin with no `archive/<slug>/*` tag covering its tip.
4. After `Add` returns it spawns the pair push the way the Worktree-Create row does, then runs the run-scope reconcile again, which now judges the pair's branches, and resumes on its verdict.

Bound: a pass may now re-create a task worktree with no operator step.
It never re-creates a pair whose weft tip an archive tag covers, which is a pair a removal tore down on purpose, nor one origin reads as owned by another machine, ended or stopped, nor one whose records worktree directory still stands, which `Add` refuses.
`AdoptLocal` lets it take over the slug's own unmarked local warp branch, the branch the run itself made.

### D2 — A pair that cannot be re-adopted fails the run once, with a way forward by row

D1 ends in a failed stop only on a permanent answer: `Add`'s not-live sentinel, or an `Add` refusal that is neither a transport or probe failure nor the teardown-lock wait of D7 running out.
A transient answer refuses the pass as today, so dobby retries and its failing-pass notice applies.

On a permanent answer, for a `running` status, the pre-run writes a failed stop through `shedengine.WriteFailedStop`, commits the status and spawns the detached status push, as a batten status commit does elsewhere.
A Busy answer from the run lock refuses the pass as transient.
Its error names the cause, `Add`'s refusal text when there is one, and its way forward depends on the row:

- At a row before Land: `lym batten cancel <slug>`, which clears the run's record and claim.
- At Land, whose work has already reached the parent: `lym fabric add <slug> --adopt-local` then `lym batten run <slug>`, never a cancel, which would release a landed entry.

The pass posts one notice to the run's parent naming the slug and the way forward, through the notice path a batten hand-back uses, and exits non-zero.
A status other than `running`, reached only by a hand pass, keeps its state and the pass refuses with the same error and way forward.

Bound: the pre-run gains one status write, `failed` with a way forward, only when the task worktree is absent and the answer is permanent.

### D3 — A way forward naming `lym fabric remove` on a batten run's slug also names `lym batten cancel`

Every stop or refusal that names `lym fabric remove <slug>` as a way forward for a slug that has a batten run record names `lym batten cancel <slug>` after it, as the step that clears the run's record.
That covers Finalize's stop when origin's parent already holds the task's work, in `internal/landingshed/finalize_owner.go`.
`lym fabric remove`, after it removes the pair of a slug whose batten run record stands and is not `done`, reports a warning in its result naming `lym batten cancel <slug>`, and changes no batten record itself.

Rationale: with the pair and the child's status gone, cancel's reached-parent test in `internal/battencli/cancel.go` reads the child as not found and returns false unless batten stands at Land or Worktree-Teardown, so the cancel clears the record.

### D4 — A teardown pushes the weft tip before the archive tag that names it

In `fabricengine.Topology.Remove`, under the pair's exclusive teardown lock and after the pending records are committed, the archive step:

1. Reads the weft branch's origin tip.
2. When origin lacks the local tip, pushes the weft branch alone, rebase-free, under the records repository's remote-refs lock; the code branch is not pushed, and no failed-push record or notice is written, since the branch is being removed.
3. Pushes the archive tag at the local tip, and also an archive tag at the origin tip when that tip is neither the local tip nor an ancestor of it.
4. Retries an archive-tag push whose remote line holds "missing necessary objects", within the archive-tag push's existing delays.

Any failure of the branch push is logged and the tag pushes go on, as today; it never fails the removal.
The branch push passes the same pause gate the tag push already passes, so a covering pause holds `Remove` at the point it holds it today.
A pair with no weft origin, or a local-only pair, pushes no branch.

Rationale: a detached push of the pair that starts while the teardown lock is held exits without pushing, so the teardown is the last chance to publish the weft tip; archiving a diverged origin tip too leaves no tip on origin that `Add` would read as live.

Bound: `Remove` waits on one more network push, a departure from the direction of the board note `fabric-remote-async` that only the teardown itself waits on.

### D5 — The tackler checks the remote before judging a commit or branch absent from it

The tackler stencil, `contracts/stencils/shed/shed-template-tackler.md`, gains a rule: a verdict that rests on a commit or branch being absent from a remote first runs `git ls-remote <remote> <ref>` and compares its answer with the commit the stop names.
A commit the remote holds is never read as absent from it.

### D6 — Every deletion of a slug's weft branch on origin is leased on the origin tip the deleter read

The weft branch's deletion on origin, in a removal's weft teardown, in its origin-only path and in the cleanup sweep, uses the leased remote delete the task branch's discard already uses.
The lease is the origin tip the deleter read under the slug's teardown lock, the tip D4 archived.
A delete the lease refuses keeps the branch on origin and is reported in the result as kept because origin moved during the removal, never as a failure of the removal.

Bound: a branch kept this way was pushed by another process between the read and the delete, which under D7 is no new pair of this slug on this machine; it is another machine's live branch, and `Add` adopting it is correct.

### D7 — A slug's pair is made, removed, discarded and swept under its teardown lock

- `fabricengine.Topology.Add` takes the slug's teardown lock exclusive from before its origin probes to its return, waiting as long as `Remove` waits; a wait that runs out refuses `Add` before any mutation, naming the slug's running push or removal and the re-run.
- `fabricengine.Topology.DiscardTaskBranch` takes it exclusive across its archive and remote delete, with the same wait.
- The cleanup sweep takes it with one try per slug before that slug's remote deletes; a held lock skips the slug and reports it skipped.
- The callers of `Add` spawn the new pair's push after `Add` returns, as today, so the push child takes the lock shared once `Add` has released it.

Rationale: no new pair's origin probes or first push interleave with the removal, discard or sweep of the slug's former pair.

Bound: `Add` and the discard may wait up to the teardown wait; neither waits on another slug's work, and fetches and origin judges stay outside the lock.

## Constraints

- The cancel's remote deletions run synchronously before `lym batten cancel` returns, and the push it spawns is the prime's records push, which deletes no branch (`internal/battencli/cancel.go`, `internal/fabricengine/remove.go`).
- A detached push holds the slug's teardown lock shared and exits without pushing when a removal holds it exclusive, and a removal waits a bounded time for it (`internal/fabricengine/teardownlock.go`).
- `Add` reads an origin weft tip as covered, and so not live, only when an archive tag's target equals it or descends from it (`internal/fabricengine/remoteleftover.go`).
- The archive-tag push takes no remote-refs lock, as `PATTERN-remote-refs-lock` allows for a tag push; D4's branch push writes a tracking ref, so it runs under the lock.
- `PATTERN-push-both-sides` keeps per-transition status pushes detached; D4's push is a teardown's push, and D2's status push is the detached one.
- `PATTERN-owner-machine`'s Worktree-Teardown exception stays as it is; D1 keeps the reconcile first for every other resume.
- Branch names carry no run id, so successive runs of a slug share the code branch and the weft branch.

## Auto-mode assumptions

Every pick is an auto-pick recorded in the support log's question ledger.

## Open risks

- The 2026-11-03 failure, a new run that found its weft branch gone from origin at Seed-Child, is not reproduced from the code. If its trigger was the new pair's first detached push still pending when a later pass judged origin, D6 and D7 do not cover it; the run's mutation records and logs, which the parent may hold, would settle it.
- The cause of the 2026-11-02 "missing necessary objects" rejection is not pinned: a plain tag push sends the objects the remote lacks. D4's branch push first and its retry cover a remote slow to see objects, not an unknown remote fault.

## Acceptance criteria

- D1: a resume at Run-Shed whose task worktree is gone and whose weft branch is on origin with no archive tag covering it re-creates the pair, spawns its push and resumes on the second reconcile's verdict; the status stays `running`.
- D1: the same resume with the slug's unmarked warp branch still local and its worktree registration stale prunes the registration and re-adopts the pair.
- D1: a resume whose task worktree is gone for an entry origin reads as owned by another machine re-creates no pair and refuses as an owned-elsewhere resume.
- D1: the run-scope reconcile of a slug whose pair worktree is absent reports the pair absent and the entry's verdict, and returns no error.
- D1: a continue pass whose task worktree is absent is refused.
- D2: a resume at Run-Shed whose task worktree is gone and whose weft tip an archive tag covers commits a `failed` status whose error names `lym batten cancel <slug>`, posts one notice, and dobby plans no further pass for it.
- D2: the same resume at Land names `lym fabric add <slug> --adopt-local` and `lym batten run <slug>`, not a cancel.
- D2: a resume whose re-adoption fails on an unreachable origin or on D7's lock wait refuses the pass and leaves the status `running`.
- D3: Finalize's stop for origin's parent already holding the work names `lym batten cancel <slug>` after `lym fabric remove <slug>`.
- D3: `lym fabric remove` of a slug with a `running` batten run record warns naming `lym batten cancel <slug>`; one with no record, or a `done` one, does not.
- D4: a removal whose weft tip origin lacks pushes the weft branch before the archive tag, and origin holds the tip when the tag push runs.
- D4: a removal whose origin weft tip diverges from the local tip pushes an archive tag at each tip, and a later `Add` of the slug reads the pair as not live.
- D4: an archive-tag push refused once with "missing necessary objects" and accepted on retry archives the tip and the removal succeeds.
- D4: a failed weft branch push still pushes the tag and the removal succeeds.
- D5: the rendered tackler prompt carries the `git ls-remote` rule.
- D6: a removal whose weft branch on origin moves between the remover's read and its delete keeps that branch and reports it kept; one at the read tip is deleted.
- D7: `Add` of a slug while a detached push of the slug's former pair holds the teardown lock waits until the push ends, then makes the pair.
- D7: `Add` whose wait runs out refuses before any mutation, naming the running push or removal and the re-run.
- D7: a cleanup sweep that finds a slug's teardown lock held skips that slug's remote deletes and reports it.

## PATTERN changes

- `PATTERN-slug-remote-order` — adds: Touching a slug's pair or its origin branches: under fabricengine's per-slug teardown lock, shared for a detached push, exclusive for Add, Remove, the task-branch discard and the cleanup sweep; a weft branch delete on origin is leased on the origin tip read.

## Notes for the plan writer

- D4's branch push pushes the weft branch alone; the coalesced pair push would carry the code branch along and write failed-push records, which a teardown does not want.
- D2's status commit and push can follow the batten status commit in `internal/battencli/commitstatus.go`.
