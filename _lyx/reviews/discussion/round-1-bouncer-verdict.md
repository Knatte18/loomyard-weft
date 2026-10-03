---
verdict: APPROVED
rationale: "All eight round-1 findings are fixed in decision-record.md (D1 entry-only aliases with a harvest bound, D6 parse-error CONTINUE and bidirectional departures with no downward BLOCKING relabel, D7 pending/settled accept, agent-shell bound and budget Reason, round-4 facts criterion, reworded Auto-mode assumptions), and the fixes introduce no unbounded edge or new inconsistency"
---

The round-1 review carries no `## Focus departures` section, so there is no departure to ratify or reject.

Finding by finding, against the current `decision-record.md`:

- **F1, legacy aliases at harvest.**
  D1 now limits the aliases to a verdict on disk at Call entry (the clear and the replay) and states the bound: a legacy word harvested from a judge this Call spawned or attached never settles.
  The acceptance criteria carry both halves: the parser criterion and the "legacy `APPROVED` from this Call does not return Done" criterion.
- **F2, parse-error latest round.**
  D6 has a bullet making a parse-error latest round `CONTINUE`, with `CONVERGED` unreachable, and states that a parse-error row is no evidence for circling.
  An acceptance criterion checks that the rendered prompt states it.
- **F3, departure direction.**
  D6 now allows a relabel in either direction, gives an example each way, forbids relabeling a BLOCKING finding downward, and requires the rationale to name every departed ID.
  The Open risks line now matches D6.
- **F4, accept state model.**
  D7 defines pending (written by the verb) and settled (written by the Bouncer before Approve), says only pending settles, and D1's Call-entry sentence covers a settled accept.
  Acceptance criteria cover re-entry and the Commit-seam-failure resume.
- **F5, facts fixture range.**
  The criterion now targets `round-4-facts.md` over judged rounds 1 to 3, which agrees with D3's 1..N-1 ledger range.
- **F6, agent-shell bound.**
  D7's Bound now says any shell addressing the run can record the decision, an agent's included, and that the refusal precondition is the whole mechanical bound.
- **F7, Auto-mode assumptions.**
  The D8 restatement is gone, the gating line now records the assumption (the draft doc is taken as intent), and the straddle line points at D1, D3 and D6.
- **F8, budget Reason.**
  The requirement has moved into D7's `continue` bullet and gained an acceptance criterion; Open risks no longer carries it.

A pass over the fixes for new attack surface found every new edge bounded: the aliases (D1), the fail-soft facts rows (D3 together with D6's parse-error rule), the departures (D6) and the circling verbs (D7).
