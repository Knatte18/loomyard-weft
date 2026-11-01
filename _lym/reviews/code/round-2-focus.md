---
round: 2
focus:
  - "internal/loomshed/samplersession_test.go TestSamplerSession and `wake` in internal/loomshed/samplersession.go: does a test row reach the done-flag judgments on a wake during the wait, where the flag is raised after the call starts on a live session? Settle whether the dirty-checkout branch resumes the session, mails the path and keeps waiting, and whether the damaged-record branch ends Stuck with Halt and a reason."
  - "`decisionVerb` in internal/loomcli/decision.go, the named-caller branch calling the sampler-strand read: when the sampler record or status is unreadable, is the run's parent orch still allowed to append, and is a non-parent caller still refused with a message naming a retry? Settle whether a test row in decision_test.go covers a failing sampler-strand read with the parent as caller."
---

Both entries target code changed since round 1 and ask whether the behavior is now pinned by a test.
