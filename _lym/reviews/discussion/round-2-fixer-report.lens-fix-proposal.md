```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (fix-proposal lens).

**1. MEDIUM, F27, D6's new first `budget` rule ("An escalation whose record or detail names an input that could not be read… is handed on at every level").**
- The proposal assumes the escalation names the unreadable input. Only `blockingEscalationDetail` and `liveRedDetail` write such a detail.
- `softCapRule` escalates through `b.escalate`, which passes an empty detail (`internal/shedadapters/bouncer.go:1654-1656`). It does this on an unreadable review or ledger of the round, and on an unreadable previous decision or ledger.
- The fallback, "whose round's review, ledger or live state the arbiter cannot read itself", covers only the current round. Concrete case under `true`:
  - The previous round's ledger fails to read in `softCapRule`, so it escalates `budget` with no detail.
  - The current round's ledger reads, the gate is green, and every open BLOCKING finding is deferred.
  - The arbiter accepts. That breaks the new Bound "only with every input read".
- The rule should also hand on a `budget` escalation that has no detail, or name the previous round's decision and ledger among what the arbiter reads itself.

**2. MEDIUM, F31(2), D6's rule "A `budget` escalation at or past the perch's hard cap… is handed on in place of the continue".**
- The fixer put this rule second in a list headed "in order", ahead of the accept rule.
- Read in that order, a green `budget` round under `true` at the hard cap, with every open BLOCKING finding deferred, is handed on before the accept rule is reached.
- But `circlingVerb` never refuses `accept` at the cap (`internal/loomcli/circling.go:76`). So D6 now drops an accept that its own first rule allows.
- The rule belongs where the continue is decided, after the accept rule.

**3. MEDIUM, F35, D7's new sentence "for an agent question it relays as severe, it answers the agent that the operator decides, which ends that question's chase".**
- It contradicts D9's bullet "An agent's question is answered and never relayed" under `unattended`. That bullet is unchanged, so the artifact now says both.
- Concrete case:
  - The agent gets "the operator decides" as its answer.
  - It ends its turn without output, so the row halts with `settle: orch`.
  - D10 then chases that new halt and posts it as severe one interval later, a second severe post for the same issue.
- The orch should hold the run as it does for a relayed halt, not only answer the agent. D9's bullet also needs the same exception.

**4. LOW, F26, D3's exclusion of "a `go-rule` `restep` under the interrupted-reinvoke rule", matched by "its rule".**
- A go-rule record's rule field is the free text `decision.Rule` ("an interrupted step under the reinvoke policy: re-step", `internal/sheddriver/driverules.go:296`).
- So the count matches message text, which the no-message-text principle D4 keeps forbids. D3 already gives settle records the stop's class, which could be matched instead.

**5. LOW, F34, D8's "a post with no key dedups only against an earlier keyless post".**
- Concrete case: lym posts a keyed severe halt. The orch's turn reply about the same halt then omits the key.
- That reply now mentions the operator a second time, which the old slug-wide dedup muted. D8's Bound covers only the line prefix, not a dropped key.

Calls made:
- `sed` of `bouncer.go:1650-1680`: to see whether `escalate` writes a detail, which my context lacked.
- `sed` of `passes.go:200-215`: to check how `liftPending` is consumed, for F33. Nothing reportable came of it.
