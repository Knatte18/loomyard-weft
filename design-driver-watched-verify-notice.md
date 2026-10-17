# Go driver messages the orch about a verify halt batten retries itself

`driver-watched-verify-notice` · Task · enhancement, shed, batten · running · Darn-Bouncer

## Seen

On 2026-10-16 `reed-add-parentless-placement` failed its first `publish_verify` at Publish.
The Go driver applied the rule "a verify halt matching the failure record under a watching batten: hand back", and batten sent the run back to Webster-Burler-Review by itself.
The driver still messaged the orch "Run ... stopped; stop report ...", which asks for an answer although the orch has nothing to do.
This is by design: `driveHandBack.WatchedVerifyLine` in `internal/sheddriver/drivehandback.go` exists "so the parent still hears of it", from `1c812c1f8`.

## Idea

When a watching batten owns the verify halt, the driver sends the parent no message, or at most a line that says batten retries it and no action is needed.
Batten's own notice for the second failure stays the parent's wake-up.
