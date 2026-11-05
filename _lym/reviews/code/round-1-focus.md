---
round: 1
focus:
  - "internal/shuttleengine/readiness.go `readinessState`: settle whether folding the whole events file when no signal lies past a non-zero `PromptOffset` can read a stale state from a turn before the offset (a prior turn start left unmatched, a stale turn end) and release or hold a send wrongly; compare with what the offset exists to exclude."
  - "Callers of `readRunSessionState` besides `sessionReadiness` (idle probe, pre-write guard, landing check, `Runner.SessionIdle`): settle whether they still read the offset-bounded state while the send path reads the whole-file fold, and whether the two can disagree for one run."
  - "The decision record's Goal asks for a bounded re-poll before refusing and a retry with backoff inside the seat start before counting an attempt as died: settle whether the diff's single readiness change covers the issue's failure or leaves a stated ask unmade, judged against the amended entry."
  - "Test coverage of the new `readinessState` fallback: settle whether any test in the diff exercises the case of no signal past a non-zero offset, or whether the `fakes_test.go` change only loosens a fake so the behavior is covered indirectly; name the uncovered branch (cause other than no-signal, zero offset, still-unknown whole-file fold)."
  - "`fakes_test.go` `hookStateSkillEngine.IdleSession` no longer treats `LOAD:` captures as idle: settle whether existing start-path tests still check what they did before, or whether the change weakens an assertion on the pane fallback."
  - "internal/shuttleengine/doc.go addition: settle whether the sentence matches `readinessState` behavior (cause and zero-offset conditions) and sits where the session-state folding is documented."
---

Seed focus for the single commit over the merge base: a three-file change to readiness folding.
Scope, stale-state risk and test coverage are the open questions; the other entries cover the decision record's unmade asks and the doc claim.
