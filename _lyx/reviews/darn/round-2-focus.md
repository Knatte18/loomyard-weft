---
round: 2
focus:
  - "internal/loomcli/decision.go, decisionNoticeClient and the notify closure in preRun: does a test reach decisionNoticeClient for the callers hub:orch, an empty name, an unparsable name and a named non-orch strand, and does each assert which kind of creel client is built (program for an orch or unnamed caller, strand otherwise, because internal/creelengine/identity.go refuses a program claim from a non-orch pane)?"
  - "internal/loomcli/decision.go, notify closure: when a strand-claim post fails with creelengine.ErrStrandClaimOutsidePane, is the post retried once through the program client, and is that path tested?"
  - "contracts/stencils/orch/orch-template-role.md, the board policy near the Discussion-Write refusal sentence: does it still tell the orch to send a darn run's decision by message, against the board entry's statement that both the Fixing a run section and the board policy drop the separate message step, while the refusal while Discussion-Write owns the record stays?"
  - "internal/loomcli/decision.go, decision add Long text: do the sentence bounding the append to a validated record and the clause that the bullet is checked with the record survive, and does the text stay under the long-help cap?"
---

Focus targets the committed diff over the merge base of `product.parent` and `HEAD`, which includes the fix commits for the round-1 findings.
