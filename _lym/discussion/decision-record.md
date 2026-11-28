## Goal

```markdown
## Seen

Issue #232 reported a running beam orch that lym read as down.
The landing of `beam-member-parent` (`b439f157e`) made the parent check admit every orch on the chain whether or not it reads as live,
but its commit says the cause of the down reading is not known,
and that the change does not cover the case where the orch's strand is also missing from reed's state.

## Idea

Find why a live beam orch reads as down: the liveness read, the strand's record in reed's state, or both.
Fix the read so a live orch reads as live, and make an orch whose strand is missing from reed's state read as down with that cause named.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-28): Name the down cause; no guessed fix for #232

Decision: Build (b): an orch whose state records a strand that reed's state no longer holds reads as down with that cause named, in lym orch status and in the routing's log line. Do not build (a): no evidence shows that a pane in another session caused #232. Instead, every down reading of an orch names which of reed's causes it is (pane dead or unlisted, pane in another session, foreground command is the start shell past the grace, strand missing from reed's state), in orch status and in the routing's log line, so the next occurrence shows its cause.

Rationale: The #232 hub is not on this machine, so the cause cannot be read. A fix without evidence risks counting dead strands as live; naming the cause makes the next report decisive.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
