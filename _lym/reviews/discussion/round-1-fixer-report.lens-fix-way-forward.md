```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked that each way forward added by the fix can be carried out from the state it names.

Ways forward introduced:
1. D3, the new pawl sentence: a pawl escalate halts with `LandingStopPrefix` "as today" and hands back to the orch. The orch then follows that halt's way forward.
2. Acceptance criteria: "A pawl fixer `escalate` at Finalize or Parent-Align halts with `LandingStopPrefix` and hands back to the orch."
3. D4 and acceptance criteria: a driver that starts or re-checks at a status-built escalate stop sends it to the arbiter.
4. D5: the orch stencil keeps "escalated a decision" for pawl, and the fixer report spec says a pawl escalate goes to the orch.

Findings:

- MEDIUM. The changed text is D3's pawl sentence ("on false the escalate halts with `LandingStopPrefix` as today") and the pawl acceptance criterion. The pawl hand-back keeps today's halt text, and its way forward names a step that cannot be carried out on a pawl run.
  Evidence: `internal/landingshed/fixerrun.go` `haltReason`, case `fixerEndEscalate`, sets the way forward to "add the deciding decision with `lym loom decision add` in the task worktree, then `lym loom resume`".
  On pawl that verb refuses with "the run has no decision record" (`internal/loomcli/decision.go:653-654`).
  `decisionWayNoRecord` (`decision.go:100`) only points at Discussion-Write and Darn-Write, and no pawl row writes either.
  So the orch that receives the hand-back meets a refusal whose own way forward does not apply to pawl, which breaks `PATTERN-refusal-way-forward`, a Constraint the record names.
  The fix made this explicit design ("as today") and the review's F2 already cited the refusal.
  So D3 should give the pawl escalate halt a way forward that pawl can carry out, for example: settle the conflict on the task branch or record the call as a board note, then `lym loom resume`.
  The alternative is a Constraint saying the pawl text changes.

- No other finding. Way forward 3 is reachable, because `stopFromStatus` sets `Error` and the driver feeds it.
  Way forward 4 is consistent with ways forward 1 and 2.

Tool calls:
- I grepped `decisionWayNoRecord` in `internal/loomcli/decision.go`, because my context showed the refusal call site but not the text of its way forward.
