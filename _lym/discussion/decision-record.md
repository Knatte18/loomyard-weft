## Goal

```markdown
## Seen

`slack-relay` landed (`5e9d4ffa`) without the operator's presence decision of 2026-11-05: presence changes only through `lym slack away|back`, the Slack texts and `lym slack stop`.
The Slack rebuild `slack-push-channels` (`92280f7e`) gives each orch its own channel and one Socket Mode relay per machine; presence is still per hub, and reed shows neither presence nor whether the relay is connected.

## Decision (operator, 2026-11-05 and 2026-11-08)

Two toggles on reed's status bar, beside `follow`, each lit when on, each redrawn when its state changes by any route:

- A connection toggle, lit while the machine's Slack relay is connected. A click starts or stops the relay, so Slack can be turned off without touching presence.
- A presence toggle, lit while presence is away (`slack`): the orchs forward everything for the operator to Slack and reach the operator there, not in the console. A click switches presence either way.

Presence is per machine, not per hub: away on a machine holds for every hub and orch on it (operator, 2026-11-08).

Presence also switches by itself:

- A message from the configured operator on Slack sets presence to away when the relay is connected and presence is `terminal`; lym sets it (the relay, before it forwards the message), not the orch's judgment, and the `[slack presence]` notice goes out as for any change.
- A line the operator types in an orch's pane while away sets presence back to `terminal`, set by lym from the `UserPromptSubmit` event shuttle already records.
- lym tells the operator's own lines from what lym puts in the pane itself: a slash command such as ORCHWATCH's `/compact` and `/color`, a text creel or another lym program injected (creel journals each inject), a task notification Claude Code submits itself, and mail a hook hands the session never switch presence.
  The design names how each kind is recognised, such as matching a submitted prompt against the inject creel journaled for that pane just before it, and tests each kind.

## Reaching the operator while away (operator, 2026-11-08)

The orch need not keep track of presence: lym carries its answers to Slack.

- A Stop hook on every orch session forwards the turn's final reply to that orch's channel while the machine is away; a turn that ends with no text sends nothing.
  Only the final reply goes, not the text written mid-turn; turns on automated notices, such as a landing, are forwarded too.
- Inbound Slack mail reaches the orch as creel mail, as today.
- The orch can still send to Slack on its own through creel, such as when the operator asks for something to be sent there; the shipped orch text keeps one plain line on how, and the rest of `role-slack.md` that a hook now covers is removed.

## Labels (operator, 2026-11-08)

The connection toggle reads `slack`, the presence toggle `away`.
```

## Scope

## Decisions

### Added after Discussion (operator, 2026-11-08): Slack sender names carry the machine

Decision: When lym posts to Slack under a sender's name, the name carries the machine in the cross-machine address form <machine>::<sender>, such as neve::ly:orch, so the operator sees which machine wrote. The same :: form is the address for cross-machine creel mail (slack-machine-bridge).

Rationale: Operator, 2026-11-08: the app showed ly:orch with no machine; :: was chosen earlier over @ (looks like e-mail) and brackets (shell globbing).

### Added after Discussion (parent, 2026-11-08): The Slack state job must never run the test binary: it froze the machine twice

Decision: Commit 4aa372a7 pins a tmux status job built by composeSlackStateCommand from executablePath(), the absolute path of the running binary. Under go test that binary is reedengine.test, so every tmux session a reedengine test creates runs 'reedengine.test slack bar' on each status refresh; the test binary ignores the arguments and runs the whole suite again, which creates more sessions with the same job. That fork bomb (hundreds of reedengine.test, tmux and bash processes, load above 800) froze Neve at 14:37 and again at 15:40 on 2026-11-08. Before any further gate or test run: (1) the status job and the toggle click must never invoke a test binary: tests inject an executablePath stub that returns a harmless command or none, and production resolves the lym binary as the other reed buttons do; (2) add a guard so a reedengine.test started with unexpected non-test arguments exits at once (TestMain), and a test that proves a pinned status job in a test session does not launch the test binary; (3) check every other place this change pins a command from executablePath (slack toggle click, hooks in claudeengine settings) for the same loop. Run only targeted package tests until (1) and (2) are in.

Rationale: Journal memsample on Neve, boots -2 and -1: by_program reedengine.test(125) at 14:36:46 rising to 537, and 176 to 517 at 15:40-15:41; both starts minutes after this run's writer committed or resumed.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
