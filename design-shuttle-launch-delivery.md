# A shuttle session gets its prompt and its creel hooks at launch

`shuttle-launch-delivery` · Task · bug, shuttle, claude, creel, darn, loom

Two faults in how a Claude session started through `lym shuttle` gets what it needs at launch.

## Part 1: Darn-Write never received its shuttle prompt (issue #42)

The Darn-Write session's first message only asked it to load the scribe skills and reply ok. The shuttle prompt carrying the darn writer's instructions (`.lym/shuttle/<id>/prompt.md`) never followed. The session sat idle until the operator typed a nudge, then found the task through `_lym/shed/<slug>/status.json` and the decision record, and made the change without the instructions. The operator later pasted the prompt path by hand. Only then did the session learn to check the board, commit and write `_lym/landing/summary.md`. By then it had already run `lym gate test` itself, which the prompt forbids.
The loom history recorded Darn-Write as done at 19:31:52Z although the prompt was only delivered by hand.
Provenance: task `transitional-report-note` on the `cs` hub.

Wanted: find why the shuttle prompt was not delivered after the skill-loading first message (ordering or timing of the prompt send versus the session start hook). Deliver it reliably, or have loom detect an idle session that has not received its prompt and send it again.

## Part 2: a `lym shuttle open` session gets no creel hooks

The trial agent `ly:cs-trial-2`, opened with `lym shuttle open --role cs-trial-2`, got every message from `ly:orch` as a reminder line, `[creel] 1 unread: #611 from "ly:orch" — run lym creel read`, never the message itself.
An opened session's launch carries no creel hooks, so creel marks its strand without `CreelHooks` and treats it as a pane-mail strand (`creelengine/doc.go`, Mail).
On that path mail from an agent strand is never typed into the pane, by design, so only the reminder reaches the agent.

Wanted: a session opened with `lym shuttle open` in a hub is a full lym agent: its launch carries creel's hook delivery (the tool-boundary claim, the turn-end Stop delivery and the idle waiter) and its strand is marked `CreelHooks`, so mail reaches it the way it reaches a run session.
