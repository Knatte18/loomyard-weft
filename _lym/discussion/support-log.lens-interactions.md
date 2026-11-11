```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked each pair of decisions that act on the same state.

**Findings**

1. MEDIUM — D5 against the review row's timeout (D2's purpose). The parent deadline is never capped below the chair's own `review_timeout_min`, so a chair still waiting on the parent can be killed. The round then fails instead of going on without the parent.
   Evidence: D5 says only "the review row's own timeout still applies". The assistant has this exact cap: `assist_reviewer_timeout_min` is "capped so the assistant ends before the reviewer's own timeout, review_timeout_min" (template.yaml line 36). D5's 30-minute default is added on top of the chair's own review time, and no validation ties the two keys together. Fix: cap the deadline the way the assistant's is, or validate it against `review_timeout_min`, and add a criterion for it.
2. MEDIUM — D4 against D5, on the ask state. Both verbs write the same state with no compare-and-set.
   - Evidence: an `answer` that lands at the deadline and an `await` that marks `timeout` at the same moment can overwrite each other.
   - One bad outcome: the parent report is written, but the state reads `timeout`, so the chair drops a real answer.
   - The other: the state reads `done` after the chair already consolidated without the parent, so the copy step (D4) copies a report the round never used.
   - The record says nothing about locking or ordering. Fix: the state moves only from open, under a lock or an atomic check of the current state, and a test races the two verbs.
3. LOW — D7 against D1, on the review step's marker. D7 puts the parent part "beside the assistant's part", but `burler-step-review.md` has a single optional `assist_rules` marker. A round with both seats needs the two parts concatenated or a second marker. The record leaves this to Plan, and nothing in it contradicts either way.
4. LOW — D2 against a `run_settings` change in mid-perch. When `discussion_review_ask_parent` flips to false after round 1 had parent findings, it is unstated whether round 2 still asks. D2's "with it on" suggests it doesn't, which leaves the parent's findings unconfirmed, and no criterion covers this.
5. LOW — D6 against D7 and the chair's round. D6 has the fork record calls with `lym loom decision add` "before the answer", so the record changes while the chair is reviewing it. The chair's review may then flag something the call already settled. Existing decision-add behaviour mid-round is the same, so this is not a new break, but the record doesn't mention it.

**Pairs checked**

- D2 and D3 (which rounds ask, and the per-round ask record): held.
- D2 and D5 (a timeout or `no-parent` in round N, then no ask in round N+1): held. It is consistent with the D2 criterion, though a lapse drops the parent for the rest of the perch.
- D3 restart and D5 deadline (a restart past the deadline makes `await` mark `timeout` at once): held.
- D3 `no-parent` and D5/D7 (the chair writes its "did not arrive" line): held.
- D4 and D5, on the ask state: broken (finding 2).
- D4's late-answer refusal and D2's no ask next round: held. The parent is pointed to `decision add`.
- D4's copy and D5's timeout: broken by the race only (finding 2).
- D5 and the review row timeout: broken (finding 1).
- D6 and D4 (the fork runs the verb from the orch's worktree, as `decision add` is run): held.
- D6 and D7 (calls recorded mid-round): held, with the LOW note (finding 5).
- D7 and D1 (`origin: parent` next to `assist:assist`): held. Origin is free text in `verdict.go`.
- D7 and D1, on the shared stencil marker: LOW (finding 3).
- D7 and the Bouncer/escalate path (parent findings are fix-class, so no loop back to the parent): held.
- D2's bound (`auto_accept`, hard cap) and D7's "keep every finding": held. Accepting with findings open is stated.
- Acceptance criteria against D2 to D7: they match the decisions. No criterion covers finding 1, 2 or 4.
- Constraints (lym-files, tolerant-records, daemon-wakeups) against D3 to D5: held. PATTERN-tolerant-records does not cover concurrent writers, which is finding 2.

**Calls**

- I grepped `burler-review-format.md` and `burlerengine` for `origin`, to check whether `origin: parent` would be refused. My context held only the assistant's `assist:` origin.
