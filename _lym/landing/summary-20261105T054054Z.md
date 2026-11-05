# Shuttle refuses a hidden anchor; Codex sessions read their turn state

Two shuttle faults from the Codex trial of hooks, shell guards and skills.

## A hidden anchor is refused before any strand exists

`lym shuttle run --anchor hidden` always ended died after its start retries, on Claude and Codex alike.
The fault was in neither engine: reed registers a hidden strand without starting its command or giving it a pane, so the provider never started and shuttle's startup probe read the paneless strand as died.
A shuttle run needs a running provider, so the hidden anchor cannot work for one.

- `Spec.validate` in `internal/shuttleengine/spec.go` refuses `render.AnchorHidden`, naming why and `own-window` with `KeepViewer` as the way forward.
  The refusal comes before any run directory or strand exists, so no Go caller can reach the dead start either.
- `lym shuttle run` accepts only `--anchor own-window`, and its refusal for any other value names the hidden case and `--focus=false` as the way to keep the viewer where it is.

## Unproven liveness no longer hides the turn state

`lym shuttle state` read every Codex run as `unknown/liveness-unproven`, and every Codex wait logged `shuttle: session state disagrees`, because `SessionFold.settle` ranked an unproven liveness above the signal-derived state.
`codexengine.Codex.ProcessLiveness` answers unproven for every session, since Codex keeps no session registry, so the derived state never showed.

- `settle` in `internal/shuttleengine/sessionstate.go` no longer has an unproven case.
  An unproven liveness settles nothing: a dead process, an unreadable events file and an empty signal history still outrank the derived state, as before.
  A Codex strand now reads busy while a turn runs and idle-stalled with cause interrupt after an interrupt.
- `SessionCauseLivenessUnproven` is removed, since nothing produces it.

## Design decisions

- The hidden anchor is refused rather than made to run.
  Making it run would change reed's hidden-strand contract, which parks a strand without a process until it is surfaced, and that contract serves `lym reed add --anchor hidden`.
- The readiness reading behind `Runner.SessionIdle` and the verified send's idle wait now follows the hook-derived state for any engine whose prober answers unproven, Codex among them, where it used to fall back to the pane alone.
  This follows from the fold change: the fold is the one source of session state for both readers.
- The hidden-strand branches in `checkLivenessTick` and Attach's candidate verdict stay.
  A run started by an older binary may still carry a hidden anchor in its record, and the branches classify it as before.

## Tests

- `TestSessionFold` rows pin that an unproven liveness leaves a running turn busy, an interrupted turn idle-stalled on the interrupt and a fresh session idle-fresh, while an unreadable events file still outranks the signals.
- `TestSpec_Validate` pins the hidden-anchor refusal, and a `TestRunCLI_Run_FlagValidation` row pins the CLI refusal.
- Readiness tests that used an unproven liveness to force the pane fallback reach the unknown reading through a run record naming an unreadable events file.
  Gate re-prompt rows that held a send on a busy pane hold it on operator activity, since a busy pane no longer outranks a hook turn end there.
  Three send rows "under an unknown reading" are deleted: an unproven liveness was their only route to that reading, and without it each duplicated its hook-reading neighbour.
