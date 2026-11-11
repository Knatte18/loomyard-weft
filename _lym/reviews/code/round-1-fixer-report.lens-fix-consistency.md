```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set for consistency with its surroundings (the `fix-consistency` lens).

**Findings**

1. LOW — `internal/slackengine/relay.go`, `RunRelay` doc comment (line 158: "runs the hub's relay until ctx ends, and returns nil when it ends or another relay holds the lock").
   - The fix added a third exit: `wake` sets `w.exiting` when no button is left on, so `RunRelay` returns nil.
   - The `wakeLoop.run`, `wake` and `exiting` comments were updated; this twin was not.

2. LOW — `internal/slackengine/relay_test.go:33-35`, the `spareOrch` comment: "Pressing its button wakes the relay and makes no channel."
   - Retire now drops that button at the first wake, because the beam is unlisted and has no channel entry.
   - The comment hides that effect, and it is why the three retire tests had to move to `r.hint(testChannel)`.
   - The only remaining use is the start-time row near line 913, which depends on the drop not mattering.

**Checked and consistent**
- `docs/overview.md:898` says the relay runs only while some orch has its button on, and `internal/slackengine/doc.go:95-96` says the same for starts. Both match the new self-exit.
- The `retire` doc comment, the drop-on-archive sentence and `ensureForwardedChannels` agree with the new `retire`.
- `PATTERN-daemon-wakeups.md` now names the forwarding-file watch, which matches `watch()`.
- The `rootProjectHandle` const matches `P0` in the board docs, and no other `RootProjectHandle` reference is left.

**Tool calls**
- Bash grep of `slackengine/doc.go`, `start.go`, `slackcli/doc.go`, `toggle.go`, `docs/overview.md`, `role-slack.md` and `relay_test.go` for relay-lifetime and `spareOrch` mentions: my context did not hold the sibling docs.
- Bash `sed` of `docs/overview.md:895-906`, `doc.go` button lines, the `spareOrch` const and `toggle.go` stop lines: I needed the exact sibling wording.
