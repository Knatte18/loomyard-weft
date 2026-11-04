# A shuttle session gets its prompt and its creel hooks at launch

Two faults in how a Claude session started through `lym shuttle` gets what it needs at launch.

Closes #42

## Prompt delivery (issue #42)

A run whose launch defers its prompt types the skill-loading turn first and the prompt line after it.
When the prompt send was accepted but the prompt's own turn never started, the old check still read any later turn start as the prompt having gone out, so an operator's nudge hid the miss and the session never got its instructions.

- `DeliverPendingPrompt` now counts only a turn start whose instruction is the prompt line, trimmed.
  A turn start with another instruction, such as one the operator typed, leaves the prompt pending.
  It clears a stale pending-send record before it types, and types the line at most once per handle.
- `Wait` calls it at a held turn end that has no turn start read past it and no pause covering the run.
  A prompt typed there sends no hold notice, since the turn it starts is the work.
  A failed delivery is logged at Warn and tried again at the next held turn end.
- A run that `Attach` reconstructs, whose output files are missing, gets the same delivery once, at the first tick no pause covers.
  This covers a driver that died between the skill-load turn and the prompt.

## Creel hooks for `lym shuttle open`

An opened session launched with no creel hooks, so creel marked its strand without `CreelHooks` and typed only a reminder line for mail from an agent strand.

- `SessionOpener.OpenLaunch` now takes a directory of the session's own, beside the exit marker and named by the same token.
  In a hub, `claudeengine` creates it and writes a `settings.json` holding creel's hook delivery alone, and an append-system-prompt file holding the creel trust sentence.
  Both launch lines name the two files, and the launch is marked `CreelHooks`.
- The hook set is the PostToolUse mail claim, the turn-end Stop delivery and the idle waiter.
  It records nothing, since an opened session has no events file.
- `Runner.Open` passes the mark on to the strand.
  It removes the directory when adding the strand fails, and the engine removes it on a later write failure.
- An engine told no board directory writes nothing and marks nothing, as before.

## Design decisions

- Open settings carry hooks only, and `--settings` stacks over the worktree's own settings, so the session's other settings, deny rules and hooks stay as before.
- The Stop and waiter hook builders in `settings.go` are shared with the run settings (`creelStopDelivery`, `creelWaitHookEntry`), so a run session and an opened one deliver mail the same way.
- Prompt redelivery is judged by the prompt's own instruction and not by any turn start; the cost is the bound below.

## To check by hand

Not run against a live tmux session:

- Open a session with `lym shuttle open --role <role>` in a hub and post it mail from `ly:orch`; the message itself, not the `[creel] 1 unread` reminder, should reach the session.
- Start a darn run, type a nudge into its pane before the prompt lands, and confirm the prompt still arrives.

Findings a review left open:

- Redelivery at a held turn end counts as delivered: a prompt the provider queued while a turn ran starts its turn only when that turn ends, so a held turn end read before it can type the line a second time (`deliverPromptAtHold` in `internal/shuttleengine/hold.go`; the bound is written in `internal/shuttleengine/doc.go`).
- The prompt-delivery guards and failure branches may lack coverage: the covered-pause, already-delivered and failed-send paths of `deliverPromptAtHold` and `deliverPromptOnAttach`, tested in `hold_test.go` and `attach_test.go`.
