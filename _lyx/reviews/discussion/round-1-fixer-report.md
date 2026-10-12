# Fixer report — discussion, round 1

Every premise was checked against the code: the run lock in `internal/boardengine/doc.go` and `runlock.go`, `lyx loom decision add` in `internal/loomcli/decision.go`, the run states in `internal/shedengine/status.go`, the arbiter keys' `none` category in `internal/loomengine/config.go`, and the empty-deploy line in `contracts/stencils/orch/orch-template-role.md`.
Each holds.

## Changes

- F1 (fixed as proposed, with one addition): D8's board rule is split by holder.
  A decision on an entry no run holds goes into its body through `lyx board`, and a decision on a run-held task goes through `lyx loom decision add <slug> --by parent`.
  The board's run lock is stated unchanged, the Rationale is rewritten, and the `orch-split` open-risks bullet no longer claims D8 answers part of that question through the board.
  Scope's orch-stencil bullet now says decisions are recorded where the agent reads them, instead of "into the board".
  Addition: `decision add` refuses with no decision record (`decisionWayNoRecord`, and `decisionWayWriter` while Discussion-Write runs), so D8 says that before the record exists, the decision goes to the Discussion-Write session by message, as the verb's refusal says.
- F2: D6 defines the peer set once, as every board task whose status is a run status, whatever the state, the caller's own task excluded.
  D3's overlap check refers to it.
  D6's `show` refusal and Bound use the run-status wording.
  The `lyx loom peers` acceptance criterion lists a peer in the awaiting or blocked state.
- F3: D5's Bound is rewritten as proposed; the one-rework sentence is kept.
- F4: D2's Bound states the residual is no wider than today's, and what the token and the attempt and head checks defend against.
- F5: D4's fallback bullet stops a still-live agent and closes the attempt; the timeout acceptance criterion asserts that a later verdict is not applied.
- F6: D7's Rationale keeps the arbiter precedent for defaults only and gives the Guard-category reason for closing `alignment`.
- F7: New acceptance criterion for the local-layer and `run_settings` closure; the `lyx loom peers` criterion includes the no-board refusal.
- F8: D8 rewords the After landing empty-deploy line to the landing notice, or the next wake after any landing.
- F9: Support log Q4 replaces "or uncertainty" with a named question, an escalation without one being invalid.
  The finding's line 287 does not exist (the log has 92 lines); the Q4 entry is at line 84.
- F10: D6 keeps the prime form and states that from the prime it lists every task holding a run status and omits the parent's commits.

## Disputed

None.

## Deferred

None.
