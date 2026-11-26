```yaml
lens: board-scope
outcome: reported
```

I reviewed the record's fit with the board through the board-scope lens. I found no contradiction with the board, one medium design finding about how many tier1 tests the design keeps, and one question for the parent.

**Findings**

1. **MEDIUM, design, D6 and its acceptance bullets.** The entry asks for a few end-to-end tests that pin the seam in place of hundreds. The record keeps many more than a few.
   - D6 keeps one tier1 snapshot-reader test per verb, one tier1 seam test per verb and one tier1 test per action kind.
   - The acceptance bullets require "For each moved verb, a tier1 test builds a real hub…" and "Each moved verb has one tier1 seam test".
   - Fabric has dozens of verbs, among them add, adopt, clone, remove, destroy, cleanup, prune, remote teardown, merge-in, handover, takeover, pull, replay, reconcile, the anchor verbs, commit and push. That is two real-hub tests per verb plus the action-kind tests: likely around a hundred tier1 repository tests, not a few.
   - Evidence: entry body, "A few end-to-end tests that pin the seam between the two, in place of hundreds of integration tests".
   - Fix: bound the seam tests by flow shape rather than by verb, and fold each snapshot-reader check into its seam test. D6 already allows verbs that share a flow shape to share a seam test; make that the rule, not the exception.

2. **LOW, decision, Open risks.** `push-retry-not-halt` (T11) has `fabric-git-free-core` in its `depends_on`. Once this motif run ends, that slug is a Sketch, and raddle files the parts under new slugs.
   - It is unsettled whether T11 should wait for the push and origin part (part 2 of D9) or for the whole chain.
   - The writer correctly recorded this under Open risks. I raise it here so it reaches the parent as a `decision` finding.
   - Question for the parent: once raddle files the parts, does T11 depend on the push and origin part alone, or on the last part of the chain?

**Goal points of the entry**
- Git-free core that reads a snapshot and decides: met by D1, D2 and D3. The entry's "plan" is named "action list" (D11).
- Thin git side that reads the snapshot and carries out the core's decision: met by D2, D3 and D4.
- A few end-to-end tests pinning the seam: partly met (finding 1).
- How the existing tests move: met by D5, D6 and D12.
- The test imports that pull fabricengine into other engines' gate scope, and how the split cuts them: met by D7 and D12.
- Discussion settles the boundary, the order of the move and the split into tasks: met by D1, D9 and the parts D9 lists.
- Operator decision not to trim the suite again: met; trimming is in Rejected alternatives and is not built.
- Operator decision relayed by `ly:orch` that fabric's tests test fabric alone: met by D12.

**Entries, peers and parent commits checked**
- `push-retry-not-halt` (T11): no conflict. It waits for this task, and D3's rule that each verb keeps today's retry budget is consistent with T11 changing the retries later. The dependency question is finding 2.
- `hardener-study-faults` (T21): no conflict. It runs after T11, so after this chain; the record's second Open risk covers it.
- `keep-landed-run-records` (T17): no conflict. It also runs after T11.
- Peer `pr-modes-no-agent`: it edits `internal/testkit/envkit/envkit.go`, which D7 retargets. That is a text-only overlap of one file, so no finding.
- Peers `dotnet-gate-fixes` and `hardener-recipe-changes`: no conflict, since neither touches fabric.
- Parent commits: `lym loom peers` lists none.

**Decision notes**
- No `Decision for fabric-git-free-core:` note is on the board, and the record holds no `Added after Discussion` entries.

**Tool calls**
- `lym board get` of `push-retry-not-halt`: my context had only the subagent's summary of that entry, not its body.
