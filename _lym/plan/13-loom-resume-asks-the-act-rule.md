# Card 13 — loom-resume-asks-the-act-rule

**Create:**
- `plan:internal/loomcli#loomCLI.refuseByAct` -> `func (c *loomCLI) refuseByAct(ctx context.Context, out io.Writer, retry string) bool`
- `plan:internal/loomcli#TestResumeVerb_RefusesWhatTheActRuleHolds` -> `func TestResumeVerb_RefusesWhatTheActRuleHolds(t *testing.T)`

**Edit:**
- `internal/loomcli#loomCLI`
- `internal/loomcli#newLoomCLI`
- `internal/loomcli#loomCLI.resumeCmd`

**Uses:**
- `internal/loomcli/doc.go`
- `internal/loomcli#loomCLI.refuseOwnedElsewhere`
- `plan:internal/runfsm#Asides`
- `plan:internal/pairteardown#AskAct`
- `plan:internal/pairteardown#ActRefusal`

**ImpactSummary:** `lym loom resume` refuses a run the act rule answers wait or hand_over for, before it takes the bootstrap lock.

**Intent:**
Make `lym loom resume` ask the act rule first (D1).

`loomCLI` gains `askAct func(l *lymcwd.Location, slug string, aside runfsm.Asides) (pairteardown.ActAnswer, error)`.
`newLoomCLI` fills it with `pairteardown.AskAct` over the hub's prime, resolved as the other prime readers of the package resolve it; a nil `askAct` checks nothing.
`refuseByAct` asks for the task worktree's slug with the shed pause set aside, since the verb resumes a run its shed paused.
- A hand-over answers with the envelope `refuseOwnedElsewhere` writes, the kind `owned_elsewhere` and the owner, so batten's InnerRun row still reads it.
- Every other answer that is not act refuses through `pairteardown.ActRefusal` with `retry`.
- A failed ask refuses naming the cause and `retry`.
`resumeCmd` calls it after `refuseOwnedElsewhere` and before the bootstrap lock; its help says the verb refuses what the act rule holds, a cancelling or cancelled run included.

`TestResumeVerb_RefusesWhatTheActRuleHolds` (tier 0, a fake `askAct`) reaches the hand-over envelope with its kind, a cancelling refusal, a pause-intent refusal naming `lym tender resume <slug>`, and an act verdict going on to the run state.

**Commit:** 13: lym loom resume asks the act rule before it wakes a driver
