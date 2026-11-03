# Darn-Write seat halts: stencil darn-template-write has unfilled marker subagent_model

`seat-stencil-marker-unfilled` · Note · bug, stencil, shed

Imported from [issue #17](https://github.com/Knatte18/loomyard/issues/17).

## What happened

`lym loom` halted as `failed` (anomaly `producer-hard-failure`) at the `Darn-Write` producer, after Preflight and Record-Write were done and Drift-Baseline and Drift-Await were skipped.
The error from lym:

```
shedadapters: Darn-Write (seats): seat run: seatengine: seat "": fill stencil "darn-template-write": stencil: unfilled top-level marker(s): subagent_model
```

## Why it is abnormal

The seat engine filled the `darn-template-write` stencil without a value for the top-level marker `subagent_model`, so the seat could not start and the whole run failed.
The seat name in the error is empty (`seat ""`), which suggests the seat's resolved config or recipe never supplied the marker.
A missing model value should either come from a default or be reported before the run starts, not surface as a hard failure inside the seat.

## The marker stays (operator, 2026-11-03)

`subagent_model` is config on purpose: the writers' lookup subagents are not bound to one model, so the fix supplies the value and never drops the marker or hardcodes a model in the stencil.
Today it is `loom.yaml`'s `subagent_model`, default `sonnet`, and an empty value is refused at config load (`internal/loomengine/config.go`), so the hard failure means the value never reached the Darn-Write seat's fill, not that it was unset.
The run (`landing-fixer-base-skills`, landed `7cffa111`) ran on the day the marker came in (`2d319fe5`), so a binary or stencil change in the middle of the run is a likely cause; note `run-pins-its-version` covers that case.
Note `lookup-agents-through-shuttle` holds where the value is heading: a lookup agent on another engine, such as Codex.

## What to do

- Find where `subagent_model` is supplied to the `darn-template-write` stencil fill in `shedadapters` / `seatengine`, and why it was empty for `Darn-Write`.
- Make Preflight (or the recipe loader) reject a seat whose stencil markers cannot all be filled, with the seat name in the message.
- Fix the empty seat name in the error.

Provenance: task `landing-fixer-base-skills`; note `loom-halt.md`.
