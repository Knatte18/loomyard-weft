```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked pairs of the record's decisions that act on the same state, then checked the acceptance criteria and constraints against those decisions.

**Pairs checked**
- D1 × D2 (Finalize's single merge path and the removed flags, merger seam and note): held. The darn and pawl resolver path becomes the only path, and nothing D1 needs is removed.
- D1 × D3 (a Finalize escalate under the parent lock): held. The halt releases the lock through the deferred release. A re-run re-merges any parent that moved meanwhile under D1 and needs no reroute.
- D1 × D4 (arbiter `resume` with the halted row as goto, while the parent keeps moving): held. Finalize re-runs from its first step under a fresh lock. The fixer commits kept by the escalate are committed, so the "before the merge-in" clean check passes.
- D2 × D5 (overlap note removed, rubric review-input paragraph and darn recipe comments): held. Both drop the note.
- D2 × Scope "Out" (`on_reroute` edges kept): held. Darn's spent and over-cap reroute to Code-Bouncer still works, with no review input. That route never wrote the note, apart from a stale one left by an earlier large-merge reroute.
- D3 × D4 (escalate prefix and `JudgesFirst(producer, reason)`): held. Only the escalate prefix is marked, and every other blocked stop at those rows reaches the tackler.
- D3 × the hand-back readers (envelope `hand_back`, driver `HandsBack`, `arbiterstop.Check`): held. All three read `landingshed.HandsBack`, so one change keeps them agreeing.
- D3 × the escalate's attempt bookkeeping × D4 resume: held. The escalate gives its turn back (`Spent--`) and keeps the Baseline, so the re-run fixer has the same attempts and the same cap span.
- D4 × arbiter off or out of spawns: held. The loom wiring always sets `Arbiter: arbiterSeam`. `MaxSpawns == 0` and a spent budget are refusals, which hand back.
- D4 × an escalate–resume loop: held. Each goto lengthens the history, so every new stop key spawns afresh, and `MaxSpawns` per run bounds the loop.
- D4 × Webster's existing mark: held. Webster's mark ignores the reason.
- D3 × the constraint on older `LandingStopPrefix` escalate halts: held. The old prefix still hands back.
- D4 × the `PATTERN-shed-stop-chain` change: held. The new text says "by its row and reason".
- D1 × D5 (orch role text): broken. See finding 1.
- D3 × D4 (way-forward wording for the arbiter and for the orch): weak. See finding 2.

**Acceptance criteria**
- The criteria for D1 (overlap, conflict, darn, gate failure, retry): consistent with D1 and D2.
- The criteria for D3 and D4 (escalate prefix, no `hand_back`, `Check` admits it, arbiter first, hand-on, arbiter off, spent budget): consistent.
- "Every other landing halt keeps `LandingStopPrefix`": consistent with D3's bound.

**Findings**

1. MEDIUM — D5 against D1: D5's list of role-text rewrites misses a stale line.
   - The orch role stencil's parent-lock bullet (orch-template-role.md:73) says: "the released run takes the lock again at its next lock-taking row and reroutes if the parent moved meanwhile".
   - After D1, a moved parent never reroutes, so this line becomes false.
   - D5 names only the `Landing halts` section, so this bullet would be left unchanged.

2. LOW — D3's way forward is vague for the orch.
   - D3 words it as "record the deciding decision … then re-run the row".
   - The orch reaches this reason through an arbiter hand-on. Today's text names `lym loom resume`, and `PATTERN-refusal-way-forward` asks for a way forward an agent can run.
   - "Re-run the row" names no command. The text should name `lym loom resume` for the orch, while the arbiter keeps its goto to the halted row.

3. LOW — D2 asserts that darn's `ReviewInput` is "nil-is-absent" without evidence.
   - D2 says that leaving darn's Code-Bouncer with no review input is its nil-is-absent value.
   - I did not verify that a nil `ReviewInput` seam is accepted by the darn bootstrap, the Bouncer amendment path and the drift-guard tests. The existing-paths lens owns that check.

Tool calls:
- grep of `internal/shedarbiter` and `drivewiring.go` for arbiter-off handling and `StopKey`: I needed to know whether an arbiter that is off reaches the tackler instead of the orch.
- `sed` of `drivewiring.go` 255–282 and `call.go` 180–192: to confirm the arbiter seam is always wired and that off or spent ends as a refusal.
