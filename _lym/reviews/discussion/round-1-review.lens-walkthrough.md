```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case and checked the end state against the decision's purpose.

**Per decision**
- **D1**, a lone run A at Finalize: it merges the parent tip P0 with no lock, writes no gate, joins the queue and leads under the lock. Purpose met.
- **D2**, member B alive and waiting in the lock queue, then leader A dying after the move: liveness by pid and start time holds. The dead-entry removal is broken (F1).
- **D3**, batch [A, B, C] on parent P1: the take works. A member whose work the parent already holds is broken (F2).
- **D4**, squash chain c1, c2, c3 on P1, where B's head contains P0: merge-tree finds base P0 and the trees are right. The message is precomputed at join, so the flaky section is lost (F3).
- **D5**, leader gated at c2 where member B deleted a directory and its `.gitignore`: the checkout and restore leave untracked leftovers (F4).
- **D6**, prefix [A, B] landed by the guarded move, with ledger landings c1 and c2: met while the leader lives. The bound on held-back records is broken (F5), and so is the backstop after a death between the move and the ledger write (F1, F2).
- **D7**, batch [A, B, C, D] with C failing: gates go full, then c2 passes, then c3 fails. A and B land, C is `failed`, D gets no outcome, two extra gates. Met. With leader A as culprit, the same failing tree is gated again (F6).
- **D8**, the `landed` path: met. The stale-`landed` cause cannot happen (F7). A `conflict` run holds the lock through a conflict session (F8).
- **D9**, `batch_max` 0 reads as 1 and 4 takes the leader plus three: met.
- **D10**, a pull-request landing resolved at join: walked and found no break.
- **D11** and **D12**, doc text only: nothing to walk.

**Findings**

**F1 — BLOCKING, design.** D6 backstop with D2 dead-entry removal (decision-record.md:44, :82).
- Case: leader A dies after the move, before any ledger write or outcome write. B takes the lock.
- A's entry is dead and has no outcome, so D2 lets B remove it. The backstop finds chain commits only by an entry that is `landed` or by a trailer that "names a queued entry".
- If B prunes first, c1 names no queued entry. Its join-time records commit is gone with the entry, so no ledger landing can be written for c1.
- The parent then stays ahead of its upstream by a commit that is no owed tip, and `syncParent` stops every later Finalize (finalize.go:739-749).
- The record does not order the backstop before the pruning, and gives the backstop no records commit once the entry is gone.

**F2 — MEDIUM, design.** D6 backstop, D8 no-outcome path, D3 and D4 (decision-record.md:82, :106, :50-51, :61-62).
- Case: the same death, after the backstop has written B's and C's missing ledger landings.
- The backstop sets no outcome, so B falls into D8 "No outcome: leads a batch". D4 then merge-trees c3 with B's head: clean, with c3's own tree.
- commit-tree writes a second, empty landing commit for B, the batch gates again and the ledger takes a second B landing.
- Nothing routes a run whose work the parent already holds to the already-landed path. `HoldsWork` would detect it (parenttips.go:222-246).

**F3 — MEDIUM, design.** D4 message against today's flaky listing (decision-record.md:6, :41, :61).
- The entry's message is fixed at join, before any gate runs (D1). The chain is built before the batch gate.
- A failure that passes on rerun reaches `fz.flaky` but never a member's landing commit.
- Today every landing commit lists those failures (finalize.go:172-202, :246-249).
- Writing them in after the gate would change the commit the gate ran on, against the Goal. The record settles neither side.

**F4 — MEDIUM, design.** D5 detached checkout and restore (decision-record.md:67-69).
- Case: the chain holds a member that deleted a directory and the ignore rule covering build output in it.
- After `checkout --detach c_k` the leader's ignored files there become untracked. `verifytree.Verify` then returns StatusDirty and runs nothing (verify.go:359-365).
- The reverse happens on restore for a directory a member added.
- Today's merge-in removes exactly such leftovers before each clean check (finalize.go:229-230, :1467-1493). D5's checkouts have no counterpart, so the batch gate stops on a dirty tree instead of judging the code.

**F5 — MEDIUM, design.** D6 bound "a member's records commits from its join on are held back" (decision-record.md:83).
- Case: B commits its records at join. Its pair pusher runs before the leader writes B's ledger landing.
- Until that write the ledger holds no landing for B, so `HeldRecordsCommit` answers not held (ledger.go:239-261, pushanchored.go:94-106). B's join-time commit is pushed.
- Today `landLocally` holds the pair's push lock across the commit and the ledger write for exactly this reason (landingdeps.go:924-925, :948-976).
- No decision names what holds the commit from join until the leader's write, so the stated bound does not hold.

**F6 — LOW, design.** D7 when the culprit is the leader (decision-record.md:94, :98).
- Case: A fails at prefix 1, after the full gate, then c2 fails, then c1 fails.
- A then merges P1 into its own worktree, the same tree as c1. Verify has no record of failures, so it gates that tree a fourth time before the fixer runs (verify.go:375-376, :487).
- That gate is known to fail and is spent only to write the failure record the batch gate skipped (D7, decision-record.md:95).
- The rationale's "about log2(n) extra gates" leaves out this extra gate.

**F7 — LOW, consistency.** D8 second bullet (decision-record.md:104, :79).
- It names "a leader that died before the move" as the source of a `landed` outcome whose commit is not on the parent.
- D6 writes `landed` only after the move, so that cause cannot happen. Only a parent moved back by something outside the queue reaches this bullet, and the bullet and its acceptance criterion (line 169) name the wrong case.

**F8 — MEDIUM, design.** D8 `conflict` path against D1's rationale (decision-record.md:105, :36).
- Case: C gets `conflict` at chain build. When it takes the lock it runs today's merge through the resolver under the lock.
- That runs the conflict session there, up to `conflict_timeout_min`. `freeLock` frees the lock only for the fixer and the fix review (finalize.go:1216, :1303).
- The idle release keeps the lock for a run at Finalize (looplock.go:202), so every landing on the parent waits behind the session.
- D1 says the merge stays lock-free "so a conflict session never holds the lock". The one run sure to need a conflict session holds it.

Tool calls:
1. Read the code store's `post-checkout` hook, to see what D5's checkouts trigger. My context showed only that the hook exists. It only prints a warning and stops nothing, so I raised no finding.
