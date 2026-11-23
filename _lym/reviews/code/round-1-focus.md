---
round: 1
focus:
  - "internal/shuttleengine/wait.go, replyhold.go: does a held turn end always end the hold and decide afresh?
    Trace the clear at the top of the events branch against a tick that reads the turn end itself, a later tool-use event, and a turn start,
    and settle whether any path finalizes a run whose agent still awaits a reply, or never releases one that awaits none."
  - "internal/shuttleengine/wait.go: the held branch sits in an else-if chain with the gated boundary, the pause-covered boundary and the pending re-evaluation.
    Settle whether a hold can starve gate re-evaluation, an unsent reprompt or the run deadline, and whether a pause covering the run leaves the hold in a wrong state."
  - "internal/creelengine/verbs.go Store.AwaitsReply: check the predicate against the message lifecycle fields
    (Replied, Unanswered, EndedUnreadBy, Holder, Status, ReplyTo).
    Does it count a question a removed or program holder keeps, a reply already read, or a message of another hub or machine?
    Is the nil-original and zero ReplyTo case safe?"
  - "internal/creelengine/server.go, protocol.go, internal/creelcli/goclient.go: does awaits-reply take the strand from the request From field
    safely under the daemon's sender checks, and does any strand or program signing reach another strand's questions it should not?
    Is the verb's only-Go-client claim true?"
  - "Runner wiring in internal/battencli/drive.go, burlercli/wiring.go, loomcli/wiring.go, webstercli/wiring.go:
    does every runner that notifies a parent get the probe, and does any run path that starts a runner skip it?
    Compare against the answer key's every-run-agent clause."
  - "Scope against the board entry: the entry asks that every run agent, the drift judge included, asks its parent by creel mail rather than in its pane.
    Read internal/driftengine/judge.go and the two parent-directive stencils against that.
    Do other run-agent stencils or mode texts still send a question to the pane, and does the change drop or weaken a stencil rule the entry did not ask to change?"
  - "Test coverage of added behavior: internal/creelcli.GoClient.AwaitsReply has no mapped test, and survivors sit in replyhold.go (the probe-failure Fail and Recover, the release statement)
    and in wait.go around the hold clear and the held-turn comparison.
    Settle which of these the diff's own contract says a test must pin, and whether replyhold_test.go and awaitsreply_test.go assert independent literal values."
  - "Commit trailers and the change description: check the Root-Cause trailer against the defect the diff removes,
    and the claims in _lym/landing/summary.md, including its Merge danger call, against the diff
    (a new creel protocol verb and a shuttle behavior change that other binaries read)."
---

Round 1 seed from the committed diff over the review range and the decision record's Goal, which holds the board entry alone.
