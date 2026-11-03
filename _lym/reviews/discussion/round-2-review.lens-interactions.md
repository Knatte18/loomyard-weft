```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked each pair of the record's decisions that act on the same state, then checked the acceptance criteria and constraints against the decisions they touch.

**Pairs checked**
- D1 × D7 (the re-adopting `Add` under the teardown lock; the pair push spawned after release): held.
- D1 × D8 (the second reconcile after a re-adopt, and tracking refs): broken for the warp side (F3).
- D1 × D2 (`Add`'s answers split into the closed permanent set and transient): broken (F2).
- D1 × D6 bound (a lease-kept branch is another machine's, adopted by `Add`): held, because D1's first reconcile refuses an entry owned elsewhere.
- D1 × D3 (`lym fabric remove` warning, then a resume answers not-live and D2 names cancel): held.
- D2 × D3 (way forward before Land is `lym batten cancel <slug>`): broken (F1).
- D2 Land restore × D1 × D8 (restored local weft, origin weft deleted by D6, re-adopt): held for the weft.
- D2 Land restore × D6/D7 cleanup sweep (restored branch has no worktree): broken (F4).
- D2 `ErrRemoteLeftover` way forward × D6 (hand archive and leased delete, local weft becomes live): held.
- D2 × D7 (lock wait is transient): held.
- D2 tag pick × D4 (two archive tags from one removal, filtered by the child's `done` status): held.
- D3 Finalize exception × D1 (owner pause blocks passes through `refuseCovered`): held.
- D4 × D6 (lease tip equals archived tip): held in function; the wording is inexact (F5).
- D4 × D7 (longer exclusive hold; `Add` and discard refused; D1 transient): held, as the D7 bound states.
- D4 × D8 (D4's branch push writes a tracking ref; D6's delete or the lease keeps it): held.
- D6 × D7 sweep (one try per slug, archive then leased delete): held.
- D7 × D8 (`Add` drops the ref inside its exclusive hold): held.
- D7 × Constraint "a detached push exits under a removal's exclusive hold": held. The push also exits under `Add`, the discard and the sweep, but no live pair of the slug exists then.
- Acceptance criteria D1 to D8 against their decisions: consistent, except the D2 first criterion, which carries F1.
- Constraints (owner-machine exception, remote-refs lock, push-both-sides carve-out) against D1, D2 and D4: held.

**Findings**

- **F1 — MEDIUM, design. D2 "At a row before Land" way forward (and the first D2 acceptance criterion).**
  `lym batten cancel <slug>` without `--remove` keeps the pair parts that made `Add` refuse. The next run of the slug then fails at Worktree-Create on the same refusal.
  - That run's ordinary `Add` has no `AdoptLocal`, so a kept unmarked warp branch refuses with `ErrBranchExists`.
  - A standing records worktree directory refuses as `weft worktree directory already exists`.
  - A kept local weft branch that diverges from origin refuses with `ErrRemoteLeftover`.
  - Only `--remove` reaches `removePairAndBranch`, which finishes the pair. D4 and D6 would then archive and lease-delete the diverged tip.
  - Evidence: `internal/fabricengine/add.go:291-293`, `:328-331`; `internal/fabricengine/remoteleftover.go:154`; `internal/battencli/cancel.go:635-659`, `:683` ("its pair was kept").

- **F2 — MEDIUM, design. D2 "Every other answer is transient … any refusal outside the closed set" (decision-record line 59).**
  D1's re-adopting `Add` can meet refusals that persist until an operator acts, and none of them is in the closed set:
  - a warp branch checked out in another live worktree (`refuseAdoptLocalCheckedOut`, `add.go:341-345`);
  - an empty own marker on the prime's weft (`add.go:255-258`);
  - the prime on a detached or unborn HEAD (`add.go:351-361`, still reached with `Parent` empty);
  - a pending anchor move on the prime's weft (`add.go:260-266`), which D2 names transient explicitly.

  Each of these refuses the pass with the status still `running`, so dobby retries every pass. That is the "fails pass after pass" outcome the Goal forbids. The closed set, or a rule for refusals outside it, needs to cover refusals that persist.

- **F3 — LOW, design. D8 scope against D1 step 3 and D2's Land restore of the task branch.**
  D8 drops a stale tracking ref only for a forked branch or an adopted local weft branch. D1 also adopts a local warp branch under `AdoptLocal`, and D2 tells the operator to recreate the task branch locally. If origin lacks that branch while a stale `refs/remotes/origin/<task branch>` survives, `originTargets` keeps it and `judgeBranch` reads it as gone. D1's second reconcile then stops the resume.
  - Evidence: `internal/fabricengine/originreconcile.go:539-541`, `:673-681`.

- **F4 — LOW, design. D2 Land restore against the D6/D7 cleanup sweep.**
  The restore creates the local weft branch with `git branch <weft branch> <tag>` and no worktree. The sweep classifies any weft branch whose warp branch has no worktree as an orphan. Under `--apply` it archives and deletes that branch, which silently undoes the restore before the re-run, and the record says nothing about this window. It is recoverable, since the tag remains, and the sweep only runs from a hand `lym fabric cleanup --apply`.
  - Evidence: `internal/fabricengine/cleanup.go:199-261`; `internal/fabriccli/fabric.go:1003`.

- **F5 — NIT, consistency. D6 "D4 archives exactly that tip".**
  Suppose D4's branch push fails and origin's tip is an ancestor of the local tip. D4 step 3 then tags only the local tip, so the lease tip is covered by a descendant tag rather than its own. The delete is still safe under the covered rule (`remoteleftover.go:77`, `destroy.go:569-575`), but the sentence overstates.

**Tool calls**
1. `grep` for `.Cleanup(` callers and `sed` of `destroy.go:565-600`: the manifest lacked who runs the fabric sweep and the archived-on-remote dirtiness rule (F4, F5).
2. `grep` in `internal/shedverbs` for `PreRun` and the run lock: the manifest lacked whether D2's `WriteFailedStop` runs while the pass holds its own run lock. I found no conflict and made no finding from it.
