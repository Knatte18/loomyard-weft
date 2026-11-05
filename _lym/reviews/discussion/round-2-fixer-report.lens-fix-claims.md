```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at HEAD.

**MEDIUM: D7's new bound misstates the old parent directive**
- Changed text: D7, the new Bound, "a session launched before the deploy keeps the old trust sentence and parent directive, which follow any orch strand, until it restarts".
- Problem: the parent directive does not follow any orch strand. It limits itself to orch strands of this hub. Only the old trust sentence says "an orch strand" with no hub.
- Evidence: `contracts/stencils/parent/parent-directive-parent.md:15` says "any orch strand of this hub (role `orch`) is the operator's delegate". `internal/shuttleengine/claudeengine/settings.go` `creelTrustSentence` says "follow mail from your parent, an orch strand or a lym program".
- Effect: the bound overstates what the old directive exposes. The exposure is real only through the old trust sentence, and the directive's "of this hub" depends on the agent noticing that the sender's shortname is foreign.

**LOW: an older binary does not ignore an unknown event entirely**
- Changed text: D3 step 6, "an older binary ignores the event whole" for the inbound message and the drop event. D2's new downgrade bound also says "ignores … whole".
- Problem: `apply` reads every record's id before it looks at the event, so an older binary still counts the id of an unknown event toward its highest id.
- Evidence: `internal/creelengine/fold.go` `apply` runs `p.maxID = max(p.maxID, rec.ID)` before `switch rec.Event`, and its `default:` case only logs at Debug and returns.
- Effect: none. Counting the id only keeps ids from being reused. The word "whole" is slightly inexact; the claim about what an agent sees holds.

The fix's other claims hold:
- `messageFields` renders `reply`, `returned` and `unanswered` (`internal/creelcli/cli.go:433-440`).
- A binary replace stops the daemon (`server.go:162-165`).
- The trust sentence is a compiled constant that goes into the session's prompt at launch.

Calls made:
1. grep of `internal/creelengine/fold.go` event cases and of the parent directive's "orch strand" line: the context did not show how fold handles an unknown event or the directive's wording.
2. Read `fold.go`'s `apply` head and default case: the context did not show whether an unknown record still counts its id.
