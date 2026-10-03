# Support log: pair-lifecycle

## Interview

Autonomous session: no operator answered; every turn below is a question the agent raised against the board entry and the code, with the answer it settled on.

1. **What does the board entry bundle?**
   Five folded items: half-removed pair after a late drive report, warp branches left on origin, reed sessions outliving `fabric remove`, the `loom start` vs self-removal race, and `main-weft` not pushed.
   Related board entries read: `orch-owns-the-loop` (one land command, a later consumer of this teardown), `orch-no-watcher`, `orch-live-hardening`, `batten-llm-friction`, `pattern-init` (depends on this task), `verify-publish-hardening`.
   None plans teardown work, so no overlap.
2. **Where does session shutdown live?**
   Batten's teardown already calls `reedengine.Down` before fabric's `Remove`; `fabric remove` does not.
   `reedengine` imports `fabricengine`, so fabric cannot call reed.
   Settled on one composite package both callers use (D1).
3. **Why did sessions survive hours after removal?**
   reed's watchdog reaps a session only once its worktree root is proven gone; the late driver recreated `<hub>/<slug>/.lyx/...`, so the root existed again.
   Ending the session first removes the writer (D4), so the watchdog needs no change.
4. **How does remove avoid killing a driver mid-report?**
   The driver commits its records before it parks or runs its teardown command.
   Remove waits, bounded, for the run lock to be free and the driver strand to be gone, dead, parked or retiring (D2); batten keeps its own grace and end-and-report.
5. **Should a refused remove still end the session?**
   No: an interactive operator pair would lose its strands to a call that then refuses.
   Refusals run as a read-only probe before shutdown (D3).
6. **How did the half-removed state arise, and how does a re-run finish it?**
   The sibling's dirtiness was checked before teardown, then the live driver wrote its report mid-teardown, so the sibling gate refused after the task worktree was gone; a re-run then failed the "worktree not found" stat.
   `Remove` now performs whatever teardown remains (D5), and batten's separate half-removed path collapses into it.
7. **How does a late stop report reach the archive?**
   The sibling-dirty refusal's own remedy was `lyx fabric sync`; `Remove` now does that commit before the archive (D6).
   After a half-removal nothing else could ever commit those records.
8. **What proves a warp branch on origin safe to delete?**
   Squash landings defeat ancestry; `checkUnlandedWork`'s merge-tree rule already handles squash for the local branch.
   Applied to the fetched remote tip with a lease (D8).
9. **How does cleanup recognise fabric's warp branches when `branch_prefix` defaults to empty?**
   A bare branch name proves nothing; ownership is an archive tag or weft branch for that slug on the weft origin (D9).
   "No open PR" comes from GitHub through `githubclient` in the CLI layer, failing toward keeping branches.
10. **What is the race in `loom start` today?**
    The detached remover waits for the `reed remove --detach` process itself, which exits at once, then removes by guid.
    `loom start` in that window sees a live driver strand and adopts it.
    reed now marks the strand retiring before spawning the remover, and start replaces a retiring strand (D7).
11. **Is `main-weft` still unpushed?**
    No: `origin/main-weft` matches the local branch at batten's latest status commit (checked 2026-10-03).
    Not advancing on landing follows from the Durable-vs-Ephemeral State Invariant (D10).
12. **What are the "sockets" in the title?**
    No socket leak is described in the body; Down already tears down the hub's tmux server when the last session ends.
    Recorded as an auto-mode assumption.

## Rejected alternatives

- **fabric calls reed directly from `Remove`.** Import cycle (`reedengine` imports `fabricengine`), and it would put loom-driver knowledge into fabric.
- **Duplicate the shutdown in `fabriccli` and keep batten's own.** Two sequences drift; batten's half-removed handling already diverged from `Remove`.
- **End the session with no quiescence wait.** Kills a driver mid-report, which is the bug that lost a drive report.
- **An unbounded wait, or batten's 900 s grace on `fabric remove`.** Hangs the hub's land step; a quick refusal is cheap to retry.
- **A `--kill-driver` flag, or overloading `--force` for live drivers.** `--force` answers dirtiness only; the documented remedy `lyx reed down` covers the rare case without a new flag.
- **Extend reed's watchdog to reap sessions whose path exists but is no longer a registered worktree.** Adds git spawns to a 5 s loop to cover a state D4 prevents.
- **Delete a stray non-worktree directory at the pair path.** It is not a registered worktree, so the destruction gate's ownership check rightly refuses it; reporting suffices once D4 prevents it.
- **Keep the sibling-dirty refusal and leave the commit to the operator.** In the finish path no verb can commit those records any more, and the refusal's remedy was the commit itself.
- **A loom-owned marker file for a retiring driver instead of reed state.** Self-removal is reed's verb, and every self-removing strand benefits; a loom marker would need its own writer in the teardown command.
- **Check the provider process's liveness in `loom start` instead of a retiring flag.** Racy in the same window, and provider specifics stay under the claude engine (Shuttle Provider-Seam Invariant).
- **Landed proof from the archived run status (Finalize done).** Fabric would read loom's status format, a layering inversion.
- **Delete origin warp branches without a GitHub check, relying on the landed test alone.** The board asks for "no open PR", and deleting a head branch closes its PR, an outward-facing side effect.
- **Make `main-weft` advance on landing.** Contradicts the per-branch weft rule; the archive tag already preserves landed records.

## Review rounds

_No rounds yet._

## Question ledger

- **Q1 (resolved, auto-pick):** Where does the shared teardown live? A new composite package used by `fabriccli` and `battencli` (D1). Rationale: import cycle, single sequence.
- **Q2 (resolved, auto-pick):** Wait bound for `fabric remove`? A 2-minute constant, refuse on timeout; batten passes zero wait plus end-and-report (D2). Rationale: never hang the land step; batten already waited its grace.
- **Q3 (resolved, auto-pick):** Do refusals precede session shutdown? Yes, as a read-only probe (D3). Rationale: a refused call must not cost strands.
- **Q4 (resolved, auto-pick):** What does a remove with nothing left return? A typed not-found error; batten maps it to done (D5). Rationale: a mistyped slug must not succeed silently.
- **Q5 (resolved, auto-pick):** Stray non-worktree directory at the pair path? Report, never delete (D5). Rationale: ownership gate.
- **Q6 (resolved, auto-pick):** Commit pending sibling records instead of refusing? Yes, through `fabric sync`'s scoped pathspec, in both paths (D6). Rationale: the refusal's remedy was this commit, and the archive must hold the late report.
- **Q7 (resolved, auto-pick):** Landed test for origin warp branches? `checkUnlandedWork`'s rule on the fetched tip, leased (D8, D9).
- **Q8 (resolved, auto-pick):** Ownership of an origin warp branch under an empty `branch_prefix`? Archive tag or weft branch for the slug on the weft origin (D9).
- **Q9 (resolved, auto-pick):** Open-PR check in cleanup? Yes, via `githubclient` in `fabriccli`, skipping warp deletion when GitHub is unreachable (D9).
- **Q10 (resolved, auto-pick):** Race fix: retiring flag in reed, or remover skips re-adopted strands? Retiring flag plus guid-scoped removal, and `loom start` replaces a retiring strand (D7).
- **Q11 (resolved, auto-pick):** `main-weft`? No code change, one doc sentence (D10).
- **Q12 (resolved, auto-pick):** "Sockets" in the title? Covered by Down's server teardown; no separate work.
- **Q13 (resolved, auto-pick):** Warp SHA for the pending-record commit's trailer when the task worktree is gone? The local warp branch tip when present, else the last recorded `Warp-SHA` on the sibling branch (D6).
- **Q14 (resolved, auto-pick, review round 1):** What happens to the retiring mark when the detached remover fails to spawn? reed clears it (D7). Rationale: the failed spawn returns an error to the requester, so the removal request never took effect and the strand keeps today's handling.
