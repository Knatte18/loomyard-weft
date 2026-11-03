```yaml
lens: interactions
outcome: reported
```

**Lens: interactions.** Each pair of decisions that act on the same state, checked with both in force:

- D1 × D2 (whether an `Add` refusal is permanent or transient): **broken**. Two refusals that only depend on local state are counted as permanent and fail the run (F1, F2).
- D1 × D7 (`Add` under the teardown lock, then the pair push spawned once `Add` returns): held. The push child takes the lock shared after `Add` releases it.
- D1 × D4 (D4's archive tags make a removed pair not live, so D1 refuses to re-adopt it): held.
- D1 × D6 (D6's bound says no same-machine pair of the slug can appear during a delete): held. The adopt-only `Add` holds the lock.
- D2 × D3 (way forward at Land vs the cancel way forward): **broken** (F3).
- D2 Land way forward × D1's not-live answer: **broken** (F4).
- D3 rationale × D2 premise (what cancel's reached-parent test reads at Land): **broken** (F3).
- D4 × D6 (which tip the delete lease uses after D4's own branch push): **broken** (F5).
- D6 × D4 (cleanup sweep and the origin-only path, where D4 does not run): **broken** (F6).
- D4 × D7 (the longer exclusive hold vs the waits of `Add` and the discard): held as transient, but outruns the wait (F7).
- D7 × Constraint 2 and the D7 acceptance criterion (a former pair's push holding the lock while `Add` runs): weak (F8).
- D2 × `PATTERN-push-both-sides` constraint (detached status push): held.
- D1 continue-pass refusal × D2 (dobby owes a full pass, which becomes a resume, then the failed stop): held.
- D7 sweep try-once × D6 sweep lease: held.

**Findings**

F1. MEDIUM, design. D1 step 3 and D2's split. D1 passes the run's anchor to `Add`. For a beam project, batten passes the anchor from the seed (`internal/battencli/wire.go:968-973`). D1 only adopts a live weft branch, and `Add` refuses `opts.Anchor != "" && weftOld.live` (`internal/fabricengine/add.go:398-401`). That refusal is neither a transport, probe or lock-wait failure, so D2 calls it permanent. Every D1 recovery of a beam project's pair would therefore fail the run.

F2. MEDIUM, design. D1 step 3 and D2's split by exclusion. A non-beam task has an empty `Parent` and forks from the prime (`internal/battencli/wire.go:957-973`). `Add` with an empty `Parent` refuses `ErrSourceDirty` on tracked dirt in the acting worktree, before any adopt logic runs (`internal/fabricengine/add.go:239-246`). A prime that is temporarily dirty would then commit a permanent `failed` status under D2, although a later pass would succeed. D2 sorts refusals into "transport/probe/lock-wait" and "everything else". The record never lists `Add`'s refusal classes, so local and fixable refusals land on the permanent side.

F3. MEDIUM, consistency. D3 against D2.
- D3's rationale says cancel's reached-parent test "returns false unless batten stands at Land or Worktree-Teardown". The code returns true only at Worktree-Teardown or past the merge. With the child status gone it returns false at Land (`internal/battencli/cancel.go:691-708`, `!childFound` at 698 comes before the Land check at 706).
- D2 rests on exactly that: "a cancel at Land would release a landed entry".
- D3 still has `lym fabric remove` warn naming `lym batten cancel <slug>` for every run record that is not `done`, Land included, and names cancel after Finalize's remove.
- So a slug at Land whose pair was removed by hand is told to run the cancel that D2 forbids there.

F4. MEDIUM, design. D2's Land way forward against D1's not-live answer.
- D2 fails at Land only on a permanent answer. Typically that is `Add`'s not-live answer: an archive tag covers the weft tip after a removal.
- The way forward is a plain `lym fabric add <slug> --adopt-local`, which is not adopt-only. On a weft branch that is not live, `Add` replaces the archived leftover with a freshly forked weft branch that has none of the child's run records (`internal/fabricengine/add.go:175-180`, `internal/fabricengine/remoteleftover.go:100-103`).
- If the local warp branch is gone, it refuses `ErrRemoteLeftover` on the origin warp branch (`internal/fabricengine/remoteleftover.go:168-175`).
- Either way the following `lym batten run` at Land does not resume the landed run that D2 means to protect. On the other permanent answers (a records worktree directory still standing, a diverged leftover), the hand `add` hits the same refusal D1 just got.

F5. MEDIUM, design. D4 against D6 (lease tip).
- D6 leases the delete on "the origin tip the deleter read under the slug's teardown lock, the tip D4 archived".
- D4 reads the origin tip (step 1), then pushes the weft branch when origin lacks the local tip (step 2). That push moves origin to the local tip.
- If the lease is the step-1 read, every removal that D4 had to push for fails its own lease. It then keeps the branch, reported as "origin moved during the removal".
- The D4 criterion "pushes the weft branch before the archive tag" plus the D6 criterion "one at the read tip is deleted" leave this case unspecified. The lease must be the origin tip after D4's push.

F6. MEDIUM, design. D6 across the deletion paths D4 does not cover.
- D6 applies the lease to the cleanup sweep and to the removal's origin-only path, and defines it as "the tip D4 archived". D4 runs only in `Topology.Remove`'s archive step.
- The sweep archives only the local tip (`internal/fabricengine/cleanup.go:243-258`, `internal/fabricengine/archive.go:84`, `resolveArchiveTip` prefers the local branch) and never reads the origin tip.
- Leasing the sweep's remote delete on an origin tip it reads lets it delete an origin tip that diverges from or is ahead of the local tip, with no archive tag covering it. That is the case D4's second tag exists to cover.
- In the origin-only path, the archived tip is the origin tip, so D6 holds there. The sweep is left with no stated archive rule.

F7. LOW, design. D4 against D7 (wait sizing).
- D4 adds a synchronous weft branch push, and keeps the archive-tag retries of up to about 62 s (`internal/fabricengine/archive.go:22-23`), inside Remove's exclusive hold.
- D7 has `Add` and the discard wait "as long as `Remove` waits", which is `teardownWait` = `StatusPushLockWait` = 30 s (`internal/fabricengine/topology.go:41`, `internal/fabricengine/pushanchored.go:20`).
- An `Add` that meets a Remove still retrying its archive times out. D2 reads this as transient, so a batten pass is only delayed. A hand `lym fabric add` or a cancel's discard is refused, and the record states no bound for this.

F8. LOW, consistency. The D7 acceptance criterion "`Add` … while a detached push of the slug's former pair holds the teardown lock waits until the push ends" against Constraint 2 and `DetachedPush`.
- A former pair's push holds the lock shared only until it finds its worktrees gone. It then exits at once (`internal/fabricengine/teardownlock.go:94-101`).
- Remove waits for any push that started earlier.
- So the criterion pins a state that lasts an instant. It does not exercise the cancel race that D7's rationale targets. Constraint 1 already shows the cancel's deletes end before `lym batten cancel` returns.

Tool calls:
1. Grepped `teardownWait` and batten's `AddOptions` call site, to size D7's wait and see what D1's `Add` inputs would be. The manifest had neither.
2. Read the `StatusPushLockWait` value and `wire.go:955-975`, the Worktree-Create `Add` call, to confirm the `Parent` and `Anchor` that D1 would pass.
