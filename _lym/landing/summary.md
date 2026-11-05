# shuttle: refuse hidden anchor; Codex state follows its turns

Two shuttle faults found in the Codex trial of hooks, shell guards and skills (`docs/research/codex-engine-trial-hooks-guards-skills-2026-11-03.md`).

## Hidden anchor

`lym shuttle run --anchor hidden` always ended as died after two start attempts.
The cause is in reed: it registers a hidden strand without starting its command or giving it a pane, so no provider starts and the readiness read finds nothing.
The fault is therefore neither in the readiness read nor in codexengine, and a Claude run died the same way.

Shuttle now refuses the anchor before any strand exists, in `Spec.validate` and in the `lym shuttle run` flag check.
The error names the cause and points at `--focus=false`, which keeps the viewer where it is.
The flag help, the command help and the `shuttleengine` package doc say the same.

Decision for review: refusing was chosen over making reed start a command for a hidden strand, which would change reed for a placement shuttle has no use for.
The task's expected outcome allowed either.

## Codex session state

`SessionFold.settle` in `internal/shuttleengine/sessionstate.go` returned `unknown` with cause `liveness-unproven` whenever liveness was unproven, ahead of the state the signals give.
`codexengine.Codex.ProcessLiveness` reports unproven for every session by design, since Codex keeps no session registry, so every Codex run read unknown and every poll logged a `session state disagrees` line.

Unproven liveness now settles nothing.
A dead process, an unreadable events file and an empty signal history still take precedence, so a Codex strand reads busy during a turn and idle-stalled with cause interrupt after an interrupt.
The `SessionCauseLivenessUnproven` constant is removed with its only use.
The package docs state the new precedence.

## Tests

The `shuttleengine`, `shuttlecli` and `battencli` tests change to match: hidden-anchor cases now expect the refusal, and fold cases with unproven liveness expect the signal-derived state.

## To check by hand

Not run against a live tmux and Codex session:

- `lym shuttle run --anchor hidden ...` exits with the refusal message.
- `lym shuttle state` on a Codex run reads busy mid-turn and idle-stalled with cause interrupt after an interrupt, and the poll log no longer shows `session state disagrees`.
