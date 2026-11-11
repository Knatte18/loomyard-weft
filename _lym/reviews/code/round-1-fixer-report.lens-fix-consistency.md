```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-set for sibling or twin text that now disagrees. Findings: none.

I checked each changed text against its siblings.

- **`creelengine/doc.go:56` and `server.go:423`:** both say "waiting inject" for stall progress, which is accurate. Waiting posts get only the warning progress, and `doc.go:103` and `protocol.go:124` already describe it.
- **`CallWatching` doc (`client.go`):** "waiting inject or post" now matches `server.go:434`, where a waiting post sends a progress response, and the `warn` callback in `cli.go`.
- **New `creelcli/doc.go` lines:** they agree with `resume()` in `client.go`. A resumed `VerbWaitReply` gets an empty `postStamp`, so there is no `receiver_live` and no warning. The final response carries no `Warning`, so the trailing `warn(resp)` stays silent, which matches "no more than once".
- **`creelcli/doc.go` ordering:** "printed before the answer line" matches the code. The progress callback and `warn(resp)` both run before the envelope is written, and the done line comes from `runVerb` afterwards.
- **Post help text (`cli.go`):** the new sentence agrees with `doc.go:47-49`. I could not confirm it stays under the help size cap. The creelcli gate passed, but I don't know that a test enforces the cap.
- **`receiver_live` omission (`cli.go`):** the omit-when-nil behaviour is documented in `doc.go:47` and handled in the `ReceiverLive != nil` branch. No twin is stale.

Calls made:
1. `grep -i progress` over `creelengine` `doc.go`, `protocol.go`, `verbs.go` and `server.go`, to find twin text saying progress is inject-only. My context lacked those files' progress wording.
