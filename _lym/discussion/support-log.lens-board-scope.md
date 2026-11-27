```yaml
lens: board-scope
outcome: reported
```

Board-scope lens: checking how the decision record and support log fit the board entry, the touching entries, the peers and parent decisions.

**Findings**

1. `MEDIUM` (class `decision`) — D4, D3 and Auto-mode assumptions: the `sync:` field and the ledger cover one hub, but the entry says "the minutes since **this machine** last had nothing owed to origin".
   - Evidence: the entry's second operator decision gives the purpose as seeing "before switching machines whether work is still only local".
   - With per-hub scope, a bar shows 0 m while another hub on the same machine still owes pushes.
   - The record turns the operator's word "machine" into "hub" by an auto-pick (Q3) instead of asking.
   - Question for the parent: should the field and the all-clear cover every hub on the machine, as the entry says, or only the bar's own hub?

2. `LOW` — D10, the goal point "no row or gate requires an agent's commits to be on origin": the record states this as a fact, but the exploration never checked it.
   - Evidence: the exploration subagent said it did not read the gate code under `gateslot`, `gatecli` and `cardverify`, "so I can't rule out a check there".
   - The point is met only if no such check exists, so the record should verify it or turn it into work.

3. `LOW` — D12 and issue #254: #254 asks to "bound the 'push is owed' retry, and record the push's final failure". D12 says this is settled because the loomcli loop is gone.
   - Evidence: the pusher's owed retry stays unbounded, with no record (D2, D3).
   - That follows the operator's decision of quiet retries with a summary notice, which wins over #254's text.
   - D12 should say so, so the change description does not claim #254's bound was delivered.

4. `LOW` — Scope Out, the catch-up fetch failure: the operator's decision says "a network outage never stops other work".
   - Evidence: the record keeps a catch-up whose fetch fails on an unreachable origin as a stop of that run's start, citing PATTERN-owner-machine.
   - The PATTERN citation justifies the exclusion and the entry's own dobby part names only the busy push lock, so this is no contradiction.
   - It is worth a line in Open risks, so the parent sees that one network-outage stop remains.

**Goal points of the entry**

- No mail per failed push; one summary past the threshold listing every owed branch; one notice when all is pushed: met (D3).
- Reed field `sync: &lt;minutes&gt;`: met (D4), with the machine-versus-hub gap in finding 1.
- Agents may push, and a failed push never stops them: met (D10). The claim that no row or gate requires origin is unverified (finding 2).
- One pusher for both pair branches, beam branches and the board, leaving operator branches alone: met (D1).
- Adaptive interval in config, 60 s doubling to 300 s, with the interval shown in the field: met (D2, D4).
- A landing pushes at once and an outage stops no other work; a local landing hands its push to the pusher; a GitHub landing waits and retries: met (D5, D6).
- A refusal stops at once with one notice: met (D7).
- #247, the periodic pusher whatever made the commits: met (D1, D2). Its "Finalize still pushes synchronously" is overridden by the operator's decisions, correctly.
- #238, the per-push mail: met (D3).
- #243 and #240, unreachable origin and GitHub EOF halting the run: met (D6).
- #218, Seed-IDE: met as settled (D12).
- #242, the merge-in leftovers: met (D9).
- Push lock busy at a dobby start: met (D8).
- The carried-over `async-run-pusher` findings: met (D11).
- #254: met as settled (D12), with finding 3.
- The hand commit forced over by `resetUnnamedTip`, with every push path the `pr-modes-no-agent` change description names checked: met (D7).
- F1 and F2 left open by `transient-stop-not-failed`: met (D6).

**Entries, peers and parent commits checked**

- `fabric-core-push-origin` (T23, depends on this task): no conflict. The ordering is the operator's dependency, and the record names it in Open risks.
- `fabric-core-foundation` (done): no conflict.
- `beam-member-parent` (depends on this task): no conflict. D6's notice goes to the run's parent and D3's summary to the hub orch; beam-orch routing stays that entry's work.
- `hardener-study-faults`: no conflict. Its teardown, cleanup and `--reset` faults are not in this record's scope.
- `keep-landed-run-records` and `area-seccomp-core`, which also depend on this task: titles only, no conflict seen.
- Peer `whole-repo-guards-off-gates`: overlap with D3's dobby tick in `internal/dobbyengine/loop.go` and `internal/beamcli/dobbydeps.go` is text-only, so no finding.
- Peers `kde-window-placement` and `weaver-bench`: none.
- Parent commits: `lym loom peers` lists none.

**Decision notes:** the board holds no `Decision for push-retry-not-halt:` note, and the record has no `Added after Discussion` entries.

**Further tool calls:** none. The board list, the task entry, the peers and the touching entries were already in my context.
