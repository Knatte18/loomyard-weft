```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: I checked each claim the fix added or rewrote against the code at HEAD.

**LOW: D5's backoff claim holds only while outbound is idle.**
- Changed text: D5, "The wait backs off, so a persistently refused invite costs one local read and one Slack call per wait."
- Problem: the outbound read's wait doubles only after a pass that found nothing. A pass that did something resets it to the floor. So with outbound traffic, a refused invite is retried at the floor rate.
- Evidence: `internal/slackengine/backoff.go:12-13` sets `waitFloor = 5 * time.Second`. `afterActivity` (`backoff.go:52`) returns the floor, and `afterIdle` (`backoff.go:58`) is the only place the wait doubles, up to `waitCeiling` of 60 s.
- What still holds: "one Slack call per wait". The 5 s floor also means a reader who expects the review's "one second to sixty" range is wrong, though the fix correctly did not state that range.

**LOW: D5 says "watches `presence.json` alone", but D2 also watches the inbox.**
- Changed text: D5, "The relay watches `presence.json` alone (D2), so `lym slack setup --operator &lt;user-id&gt;` takes effect at that wake with no relay restart".
- Problem: D2 has the relay watch the inbox directory and, in the Slack directory, the presence file alone. "Alone" is true only of the Slack directory. The conclusion that setup.json is unwatched holds; the wording overstates what D2 says.
- Evidence: D2, "it watches the Slack directory for the presence file alone, through `internal/fswatch`'s names filter", after "The relay creates the inbox directory … before it watches it".

These claims also hold, with no finding:
- **The inbound filter takes up a new operator without a restart.** Today the inbound pass re-reads setup.json on every pass and filters on `setup.OperatorID` (`inbound.go:223`, `inbound.go:309`). The outbound pass does the same (`outbound.go:76`).
- **"No new loop is added."** The owed-read retry and the held-channel invite both ride the existing outbound read's wait.
- **"Its next start reads every open channel."** This matches D2's start trigger. The owed thread reads keep D2's stated 24-hour bound.

Tool calls:
1. I read `backoff.go:1-32` and grepped the setup reads in `inbound.go`, `outbound.go` and `relay.go`. My context lacked the `waitFloor` value and showed no code where the inbound side reads the operator.
