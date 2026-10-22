# lyx loom decision add notifies the run's live agents

On the Go driver no session lives for a whole run, so a decision recorded with `lyx loom decision add` had no receiver of its own.
The orch looked up the working strand with `lyx reed list` and messaged it by hand.
The verb now tells the working agents itself.

## What changes

- `internal/loomcli/decision.go`: once the entry is committed, and on a darn run once the committed record is confirmed to hold it, the verb lists the target worktree's live shuttle runs through `shuttleengine.ReadAgentActivity`.
  It posts each one creel line naming the decision record's path and the new entry's heading, telling the agent to re-read the record's `Added after Discussion` entries.
  The line carries no decision text: the committed record stays the authority, so the line cannot pose as an instruction.
- Skipped receivers: the tackler and the arbiter, which judge stops rather than the plan, the caller's own strand, and a run whose record names no parsable strand.
- The notice signs as the `LOOM` program from the orch's pane or an unnamed shell, and as the caller's own strand from any other named caller (the arbiter), because creel accepts a program claim only from an orch pane or a process in no pane.
- The success envelope gains `notified`, the strands posted to, and `notice_errors`, one `{strand, error}` item per failed post or a strand-less item for a failed listing.
  A notice failure is logged at Warn and never fails the verb, since the entry is already committed.
- With no live run nothing is sent: a later producer reads the record when it starts, and the alignment agent at PR-Gate judges the run against it.
- The orch role stencil's Fixing a run section drops the separate message to the run's agent after `decision add`.
  The darn writer stencil names the typed line as a trigger to re-read the record.
- `docs/overview.md`, the verb's help and its file doc describe the notice.
  The help text was condensed to stay under the CLI's long-help cap.

## Design decisions

- No flag turns the notice off, which settles the board entry's open question: a decision recorded only for the record still binds the live producer, and one idle-gated line is cheap.
- Every live run of the worktree is told, not only one strand of the current producer: Go keeps no producer-to-strand map beyond the shuttle run records, and a MultiLLM row or a webster stage can hold several live sessions.
- A webster fork working in another worktree is not a run of the target worktree and is not told.

## Tests

`TestDecisionVerb_NotifiesLiveAgents` covers the receiver filter (tackler, arbiter, caller and an unnamed run skipped), the line's content, no live run, a failed listing and one failed post among several, each on a success envelope with the entry committed.
`TestDecisionVerb_Refusals` now also asserts that a refused call, the darn lost-entry refusal included, posts nothing.
