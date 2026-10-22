# loom: decision add notifies the run's live LLM agents

With the Go driver no agent lives for the whole run, so the orch had to look up the working strand and message it by hand after `lyx loom decision add`.
The verb now does that itself.
After it commits the entry, it types one short creel line into each live LLM agent of the target worktree, naming the decision record and the new entry's heading.
When no agent is live, recording stays enough: a later producer reads the record when it starts, and PR-Gate's alignment agent judges the run against it.

The success envelope gains two fields: `notified`, the strands posted to, and `notice_errors`, one item per failure with the strand and the error.

## Design decisions

- **Receivers.** Every live shuttle run of the target worktree, read through `shuttleengine.ReadAgentActivity` and the registered engines, except the tackler, the arbiter, the caller's own strand and a run with no parsable strand name.
  Go keeps no producer-to-strand map beyond the shuttle run records, so the line goes to all of them; a webster stage or MultiLLM row may hold several.
  A webster fork in its own worktree is not told; Merriam, in the target worktree, is.
- **No decision text in the line.** It carries the record path and the entry heading only, so the committed record stays the authority and the line cannot pose as an instruction.
- **Signing.** From the orch's pane or an unnamed shell the notice is posted as the `LOOM` program, which creel types in full.
  From a named non-orch caller, such as the arbiter, creel refuses a program claim, so the notice is posted as that strand.
  If creel refuses that strand claim as coming from outside its pane (`ErrStrandClaimOutsidePane`), the post is retried once as the program, as `orchcli`'s `notifyPrime` does.
- **A failed notice never fails the verb.** The entry is committed first, so a failed listing or post is logged at Warn, reported under `notice_errors`, and the exit code stays 0.
  A lost notice costs at most a rework round at PR-Gate.
- **No flag turns the notice off.** A decision recorded only for the record still binds the live producer, and one idle-gated line is cheap.
- **Delivery is creel's.** The line goes through `internal/creelcli`'s client and is typed only when the pane is idle, so a long-busy agent may see it after finishing the work the decision concerns; PR-Gate then catches the deviation.
- **Long help cap.** `decision add`'s help text must stay under `maxLongBytes` in `cmd/lyx/clitree_test.go`.
  Only the notice paragraph and the Discussion-Write refusal wording were condensed to fit; the darn-run paragraph is kept in substance.

## Docs and stencils

- `internal/loomcli/decision.go` file doc and `docs/overview.md` describe the notice.
- The orch role stencil drops the separate message to the run's agent after `decision add`, both in Fixing a run and in the board policy for a darn run.
  The refusal while Discussion-Write owns the record still sends the decision to that session by message.
- The darn writer stencil names the typed line as a trigger to re-read the record's `Added after Discussion` entries.

## To check by hand

Not run live: a real notice typed into a live agent pane.
Tests cover the verb with fake seams, including the signing split and the fallback.

- `contracts/stencils/orch/orch-template-role.md:161`: the board policy no longer sends a darn run's decision by message.
- `internal/loomcli/decision.go:446`: `postDecisionNotice` signs as the program for an orch or unnamed caller and as the strand for a named non-orch caller; `TestPostDecisionNotice_SignsByCaller` pins it.
- `internal/loomcli/decision.go:446`: a named non-orch caller outside a strand pane falls back to the program instead of failing every notice.
- `internal/loomcli/decision.go:483`: the `decision add` help text keeps its unrelated paragraphs and trims only the notice paragraph to stay under the cap.
