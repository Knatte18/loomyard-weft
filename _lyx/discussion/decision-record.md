# Decision record — decision-add-notifies

## Goal

`lyx loom decision add` tells the run's live LLM agents that a new decision is in the decision record, so the orch no longer looks up the working strand and messages it by hand.
When no LLM agent of the run is live, recording stays enough: a later producer reads the record when it starts, and the alignment agent at PR-Gate judges the run against it.

## Scope

In:

- `internal/loomcli/decision.go`: after the entry is appended and committed, one creel notice to each live LLM run of the target worktree.
- The verb's success envelope reports which strands were told and which notices failed.
- The verb's help text and file doc, and its `docs/overview.md` paragraph.
- The orch role stencil's Fixing a run section drops the separate message to the run's agent after `decision add`.
- The darn writer stencil's scope-correction section names the notice as a trigger to re-read the record.

Out:

- A new verb, a strand alias for the step's owner, and a queue holding a notice for the next producer (dropped on the board entry).
- Stops: they stay `lyx batten pause`.
- Upward messages (stops, escalations), settled elsewhere.
- The tackler and the arbiter as receivers.

## Decisions

### D1 — The verb notifies every live LLM run of the target worktree after the commit

Once the entry is committed, and the darn check that the committed record holds it passes, the verb lists the target worktree's live shuttle runs: a run whose record reads running and whose waiting process is alive.
It sends each one notice line through creel, which types it into the strand's pane when the pane is idle.
It skips the tackler, the arbiter and the caller's own strand, and a run whose record names no parsable strand name.
With no live run left, it sends nothing.

Rationale: the shuttle run records are what Go holds about which LLM sessions work for the run right now, also on the Go driver where no driver session lives; a MultiLLM row or a webster stage may hold several, and each acts on the record.
The tackler only gets a stopped step going and the arbiter judges a stop, so judging a decision's meaning for the plan is neither's job; the caller wrote the decision itself.
Bound: the new edge types one line into a live agent pane of the target worktree only, only through creel's idle-gated delivery, once per call, after the decision is committed; it changes no run state, no record and no plan.

### D2 — The notice names the record and the entry, never the decision text

The line reads that a new decision is in the decision record, gives the record's path and the entry's heading, and tells the agent to re-read the record's `Added after Discussion` entries and follow them.

Rationale: the committed record is the authority; a line that carries no decision text cannot pose as an instruction, so a receiving agent that trusts only its parent's messages loses nothing by acting on the record it already trusts.

### D3 — The notice signs as the LOOM program, or as the caller's strand when the caller is a named non-orch strand

From the orch's pane or an unnamed shell the notice is posted by a creel client signing as the `LOOM` program, so it is typed in full.
From the arbiter's pane, where creel refuses a program claim, it is posted as the arbiter's strand, which creel delivers as an unread reminder.

Rationale: creel accepts a program claim only from an orch pane or a process in no pane; a strand claim from its own pane always passes.

### D4 — A failed notice never fails the verb

The entry is recorded and committed before any notice, so a failed run listing or a failed post is reported on the success envelope under `notice_errors`, one item per failure naming the strand and the error, and logged at Warn.
The strands that were posted to are listed under `notified`.
The exit code stays 0.

Rationale: the decision is durable and binding once committed; a later producer and PR-Gate catch it even when the notice is lost, so a lost notice costs at most a rework round.

### D5 — No flag turns the notice off

Every call notifies.

Rationale: a decision recorded only for the record still binds the live producer, and one idle-gated line is cheap; a flag would add a branch with no caller asking for it.

## Constraints

- `PATTERN-creel-store`: reaching a pane goes through creel; the verb posts through `internal/creelcli`'s Go client and never types into a pane itself.
- `PATTERN-shuttle-provider-seam`: the live-run listing goes through `shuttleengine.ReadAgentActivity` and the registered engine set, naming no provider.
- `PATTERN-cwd-resolution` and `PATTERN-cliwire-sole-wiring`: the target worktree comes from the verb's existing resolver; the new seams are built in its pre-run.
- `PATTERN-refusal-way-forward`: no new refusal; a notice failure is a report, not a refusal.
- `PATTERN-test-speed` and `PATTERN-test-economy`: the behavior is pinned on the verb's existing fake-backed tests, extended with the two new seams.

## Auto-mode assumptions

- The board's open question on a flag is settled as no flag (D5).
- "The LLM producer" is read as every live LLM run of the worktree minus the tackler, the arbiter and the caller (D1), since Go keeps no producer-to-strand map beyond the shuttle run records.

## Open risks

- A webster fork working in its own worktree is not a run of the target worktree and is not told; Merriam, in the target worktree, is.
- A live agent busy for a long time gets the line only when its pane is idle, which may be after it has finished the work the decision concerns; PR-Gate then catches the deviation.

## Acceptance criteria

- A successful call with live runs posts one notice to each, except the tackler, the arbiter and the caller's own strand, and lists the strands it posted to under `notified`.
- The notice names the record path and the entry heading and holds no decision text.
- A successful call with no live run posts nothing and reports an empty `notified`.
- A failed run listing or a failed post leaves the exit code 0, the entry committed, and the failure under `notice_errors`.
- A refused call posts nothing.
- A darn call whose commit lost the entry is refused and posts nothing.

## Notes for the plan writer

- `orchcli.notifyPrime` holds the existing strand-or-program signing rule for a notice to the orch.
