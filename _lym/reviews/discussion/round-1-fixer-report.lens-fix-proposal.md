```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals that the fixer carried out as proposed. For each one, I tried to build a concrete case where the proposed change is wrong.

Proposals checked:
- F1: the seed rule and the Discussion-Write-after-ask rule.
- F2: matching the ask by seed and round.
- F3: the cap formula and `not-sent`.
- F4: `no-parent` on a nil-error return, and `timeout` for a dead orch.
- F5, D5 part: Go sets an open ask to `timeout` when the review table ends.
- F6: the YAML findings shape.
- F7: keying the orch-role section on the opener.
- F8: the key decides only whether a new ask is sent.
- F9: write `pending-post` first, then post, then set `open`.
- F10: store the posted strand and admit it.
- F11: the bound wording.
- F12 and F13: the new criteria.
- F14: the cut sentence and the new rejected alternative.

Findings:

1. MEDIUM — D5, "Once the review table ends, on success or failure, Go sets an ask still `pending-post` or `open` to `timeout`" (F5's proposal).
   Summary: a review row that fails and is restarted in the same round loses the parent review before its deadline.
   Case: the chair crashes or the table fails mid-round, and Go sets the open ask to `timeout`. `Discussion-Burler-Review` is `InterruptPolicyReinvoke` (`internal/loomshed/interruptpolicy.go:47`), so the row runs again in the same round. D3 says a restart "that finds any later state posts nothing". The new chair's await then answers ready at once, and the parent's answer, already on its way, is refused.
   This contradicts D3's criterion that a restarted row "keeps the first deadline", and F8's text that an ask already sent "is awaited and consolidated". The `pending-post` case is worse. Suppose the row crashed before posting and then failed. Go sets the ask to `timeout` on failure, so the restart never mails the parent at all, which undoes F9's "a restart that finds `pending-post` posts again".
   A fix that keeps the proposal's intent: close an open ask on a failed table only at its deadline, or when the round has ended (the Bouncer judged it, or the round moved on). Do not close it when the table merely failed. Alternatively, let a restart of the same round reopen a `timeout` that no deadline set.

2. LOW — D6, "from any sender creel stamps as one of the run's own sessions or as the hub's LOOM program" (F7's proposal).
   Summary: the key still misses the very case F7 raised whenever the driver strand does not belong to the run.
   Case: `notifyPrime` signs as the caller's strand whenever the process carries any non-orch strand name (`internal/orchcli/notify.go:197-205`). An LLM driver running the review row, such as the webster session or another driver outside the run's slug, is not "one of the run's own sessions". Its `[discussion-review request]` would then fall outside the section. The orch-role section should either key on the opener from any sender of this hub, or D3 should post with a program-signed client that ignores the strand claim (the reviewer's alternative). Only the program-signed post makes the sender fixed.

The other proposals hold. None of the cases I built for them breaks the artifact.

Tool calls: none.
