# Decision record — driver-watched-verify-notice

## Goal

The Go driver sends the run's parent no message at a verify halt that matches the failure record under a watching batten, since batten retries that halt itself and the parent has nothing to do.

## Scope

- `internal/sheddriver`: the rule table's watched-verify row stops asking for a parent line, and the hand-back's notice set drops the watched verify halt.
- `internal/loomcli/doc.go`: the Go driver section's list of messages a watched run sends.
- Tests in `internal/sheddriver` that pin the watched verify halt's notice.

Out of scope: batten's own notice for a second verify failure, the LLM driver's stencil and the unwatched verify halt's goto and hand-back.

## Decisions

### Send no message at a watched verify halt

Decision: the watched verify halt hands back with no parent message; the `WatchedVerifyLine` field and its notice branch are removed.
Rationale: batten sends the run back to its retry target by itself, and its own notice at the second failure is the parent's wake-up, so any line here only asks the orch to answer with nothing to do.
The hand-back still writes the stop report and parks the run, so the halt stays on record.

## Constraints

- `PATTERN-shed-driver`: the change stays in `internal/sheddriver`.
- `PATTERN-test-economy`: the existing rows that pin the watched verify halt are corrected, not joined by new tests.
- `CLAUDE.md`: the observable change to the driver's parent messages updates `internal/loomcli/doc.go` in the same commit.

## Auto-mode assumptions

- No message at all, over a no-action line: a line that needs no answer still ends in the driver's answer tail and wakes the orch.

## Open risks

- A watched run whose batten dies after the hand-back gets no parent message from the driver; batten's own liveness handling covers that, as for every other watched hand-back that sends nothing.

## Acceptance criteria

- A watched verify halt matching the failure record hands back, parks and writes a stop report, and sends the parent no notice.
- An unwatched verify halt keeps its one goto, and a second such halt keeps its hand-back.
- The other watched notices, a relayed parent notice, a question and a friction-failed done, are still sent.
