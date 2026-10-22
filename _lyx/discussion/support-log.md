# Support log — shuttle-open-agent-defaults

## Interview

A darn run holds no interview; every pick below is the writer's own, recorded in the question ledger.

## Rejected alternatives

- **Substitute a generated session id for `%SID%` in `reed add`**: the board's comment says an operator should never build an agent command line; fixing the help removes the need.
- **Default `--permission-mode bypass` in the `shuttle open` CLI only**: leaves interactive runs prompting, and puts claude vocabulary outside the engine.
- **Default bypass in `Runner.Open`**: the mode is engine vocabulary carried blind; the engine owns its default.
- **A separate `lyx shuttle models` verb**: the ask is to list the choices from the command itself; a flag on `open` keeps it there.
- **A configured default model for `open`**: a new config key no one asked for; the envelope already says the provider default was used.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): Do interactive runs get bypass by default too? — Yes, the operator's "every agent instance in reed" covers them (D1).
- Q2 (resolved, auto-pick): Is `prompt` still allowed? — Yes, as the explicit opt-out on interactive sessions (D1).
- Q3 (resolved, auto-pick): How does the operator list the choices? — `lyx shuttle open --list-models` (D4).
- Q4 (resolved, auto-pick): What does `open` report with no `--model`? — The default engine, with empty model and effort for the provider default (D3).
- Q5 (resolved, auto-pick): Fix `%SID%` or the help? — The help, pointing at `lyx shuttle open` (D5).
