```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked how the record's decisions act on shared state, including their failure and restart paths. I made no tool calls; everything came from the context I inherited.

Pairs checked:
- D1 with D11 (mode changes between PR-Review and Finalize): mostly held. Nothing says a recorded pull request wins once the mode has turned `off` since PR-Review ran (see F4).
- D2 with D3 and D11 (`off` under `required`): held. The row opens a pull request, and Finalize's `required` check finds the record.
- D2 with D13 (a beam's own field against its parent's mode): held.
- D3 with D4 (who skips, and where): broken (F2).
- D3 with D6 (a record left by an earlier entry, then the row Skipped): broken (F1).
- D4 with D11 (`required` with no GitHub origin): held. Finalize sends the run to `goto PR-Review`, which halts again under D4, and both halts name ways forward.
- D5 with D9 (an auth failure during the wait): held.
- D6 with D8 (the record's pushed head against the task head): held during the wait.
- D6 and D8 with D11 (Finalize's own push leaves the record's head stale): partly broken (F5).
- D7 with D8 (the agent never pushes and the row gates its commits): held.
- D7 with D9 and D11 (the session stopped at Done, with the tree possibly dirty): broken (F3).
- D7 with D12 (cancel ends the session through runproc): held.
- D8 with D9 (Done needs a pushed, gated head; a failing gate means waiting): held.
- D8 with D9 (commits added after a merge on GitHub lead to the "lacks commits" halt): held.
- D8 with D10 (the gate's verify mark and the PR mark share the driver strand): broken (F6).
- D9 with D11 (an approval of a head that Finalize's push then changes): held, since D11's bound covers it.
- D11 with the reroute path through Describe and PR-Review (re-approval needed on the new head): held.
- D12 with D6 (the close happens before teardown removes the record): held.
- Acceptance criteria and constraints against each decision: held, apart from the AC gaps named in F1, F2 and F3. The `PATTERN-github-auth` change fits D6, D11 and D12. The constraint that PR-Review takes no lock fits D8's push.

Findings:

F1 — MEDIUM — D3 with D6 and D11: a stale pull-request record outlives a Skipped row.
- A run can open a pull request at PR-Review, then reroute or `goto` back with the entry field now `off` under an `on` parent. D3 then records PR-Review Skipped.
- The D6 record stays, so D11 ("with a record…") takes the pull-request path. It merges an unapproved pull request or halts on GitHub's refusal, against the effective `off`.
- No decision clears the record, or closes the pull request, when the gate skips.
- Fix: say what a skip does to an existing record and pull request, and say whether Finalize follows the mode or the record. Add an AC.

F2 — MEDIUM — D3 with D4: the skip and halt for a hub with no GitHub origin have no owner.
- D3's resolver takes only the parent's value and the entry's field, and an `entry_gate` can only hold, skip or fail.
- D4 needs facts about the origin, the upstream and the push skip. It both skips (an entry-gate result) and halts (a Stuck, which needs the row's Call).
- Neither decision says that the gate reads the origin facts for the skip, or that the halt comes from the row's call.
- D4's AC ("an effective `on` from the parent is Skipped") cannot be met by D3's resolver as specified.
- Fix: state that the entry gate reads the origin facts for the skip, and that the row's call halts.

F3 — MEDIUM — D7 and D9 with D11: stopping the agent at Done can leave a dirty tree for Finalize.
- D9's Done arms (approved on the current head, or merged) need the head pushed and gated. Neither needs a clean tree.
- D7 then stops the session on Done, possibly in the middle of an edit.
- Finalize's clean-tree check is then Stuck with uncommitted agent edits, and the merged arm likewise.
- Fix: make Done wait for a clean tree, or say what happens to the agent's uncommitted work when the session stops. Add it to the D9 AC.

F4 — LOW — D1 and D3 with D11: the precedence between record and mode is implicit.
- D11 says Finalize "reads the D6 record and the D3 mode", but only defines `required` with no record, and "with a record".
- A record under an effective `off` is not stated. This is the same root as F1.

F5 — LOW — D6 and D8 with D11: Finalize's push is not written back to the record.
- D11 pushes the task branch but does not update the record's pushed head.
- On a reroute back to PR-Review, D8 sees the head differ from the recorded one and gates and pushes again.
- It is harmless, since verifytree skips the tree it already verified and the push is a no-op. Still, D6's record no longer means "the head the PR carries".
- Fix: say that Finalize's push updates the record.

F6 — MEDIUM — D8 with D10: the gate's verify mark clears the PR mark.
- D8 runs the `pr-review` gate inside D10's wait.
- If the gate's verify uses the `VerifyWaitMark`-style seam, it overwrites the strand's single wait mark and clears it when the verify ends.
- The `PR #&lt;n&gt;` mark is then gone for the rest of the wait.
- D10's AC ("the mark is set during the wait") breaks after the first gated agent commit.
- Fix: say that the row sets the PR mark again after each gate, or that the gate runs without its own mark.

Tool calls made: none. My context already held the record, the landingshed doc and the recipe and producer excerpts this lens needed.
