# Webster deviations list mixes in the batch's own files

`webster-deviations-list` · Note · bug, webster

## Seen (2026-10-12, `pause-esc-rewind`)

The Webster report `_lyx/webster/reports/01-running-turn-predicate.yaml` listed six `deviations`: `interruptguard.go`, `send.go`, `engine.go`, `fakes_test.go` and `run_inject_test.go` in `internal/shuttleengine`, plus `pattern/PATTERN-guarded-interrupt.md`.
The batch most likely held cards 1 to 5, so several of these probably house the batch's own target glyphs and are no deviation at all; the pair was torn down before that could be checked.

## Cause

The implementer stencil (`contracts/stencils/webster/webster-body-implementer.md`, the `deviations` paragraph) contradicts itself.
It asks the fork to list the paths it changed outside the batch's target-glyph union, then says the fork reports what it touched and never estimates the comparison itself, because `internal/planglyph`'s `ScopeGuard` makes it.
A fork cannot map a glyph to its file without that comparison, so it guesses, and the list mixes real deviations with the batch's own files.

## Idea

Pick one meaning:

- the fork reports every path it changed, and `ScopeGuard` alone decides which are deviations; the Merriam digest and the PR narrative show the guard's result, not the fork's list;
- or the fork reports nothing, and the guard's output over the batch delta is the deviation list.

Either way the list a reader sees comes from the guard, never from a fork's guess.
