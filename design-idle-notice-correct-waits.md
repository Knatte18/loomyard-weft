# The idle and held notices skip correct waits and name what an agent waits on

`idle-notice-correct-waits` · Task · enhancement, dobby, batten, creel, shuttle, loom · draft

Decision record: [Sketch](decision-idle-notice-correct-waits.md)

Depends on: [`severe-halt-slack-post`](design-severe-halt-slack-post.md), [`creel-machine-outbound-flake`](design-creel-machine-outbound-flake.md)

Filed from the Sketch `self-healing-runs`.

Takes: D11 whole: the quiet notice in `internal/dobbyengine/runnotice.go`, `readAgentActivity` in `internal/battencli/wire.go`, `Store.AwaitsReply` in `internal/creelengine/verbs.go` and `creelcli.AwaitsReplyProbe`, the held notice in `internal/shuttleengine/hold.go` and `Runner.SetReplyProbe` in `internal/shuttleengine/run.go`, and `verifyBudgetNotice` in `internal/loomshed/verifybudget.go`.
Delivers: The quiet notice raises nothing for a `Raddle-Write` run whose proposal passes with a live raddle agent or for an agent with an open question in creel, and names an unread reply; `AwaitsReply` read as two probes for the quiet notice while the held notice keeps the combined one; an idle notice names the live command, its run time and its last output, or the Finalize step and what it waits on; the held notice is not sent while a question is open or a reply unread, in every runner; the verify gate's past-budget notice names the failing runs or says none failed.
Exports: Fixed for consumers: `creelcli.OpenQuestionProbe` and `creelcli.UnreadReplyProbe`, which dobby-stall-chase's question chase reads.

The part changes these packages:

- internal/dobbyengine (about 300 changed lines)
- internal/battencli (about 130 changed lines)
- internal/creelengine (about 130 changed lines)
- internal/creelcli (about 50 changed lines)
- internal/shuttleengine (about 160 changed lines)
- internal/loomshed (about 40 changed lines)
