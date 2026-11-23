## Goal

````markdown
Imported from [issue #198](https://github.com/Knatte18/loomyard/issues/198).

The `Code-Burler-Review` seat halted the run as `producer-hard-failure` at round 1 attempt 1b, before any review ran:

```
review: stencilstore: stencil "burler-assist-consolidation" was written by lym build 8cf7f477563f044e178a070cfd7dd27cb9f03c78, which is newer than the running build 52d132a2c8e4e3b68e3e797e30d3e9ceb200ec50, and this process may not render it; way forward: restart the run's driver on the current lym binary, and update that binary first if it is older than the board
```

The run's driver process had been started on an older lym build than the one that wrote the stencil on the board.
The run had been going for over thirty minutes (Preflight at 08:25Z, halt after Code-Bouncer at 08:59Z), so the board was upgraded mid-run while the driver kept the old binary.

This is abnormal because the guard is correct but its result is a hard `failed` halt of a long-running run, with the recovery left to the operator.
Nothing warns earlier that the driver binary has fallen behind the board.

What to do: consider detecting the build skew when the driver steps between segments and halting as a resumable state, not `failed`.
Alternatively, have the driver re-exec itself on the current binary, or surface the skew in `lym loom` status before a seat needs the stencil.

Provenance: task `godocreflow-worsens-findings`; notes `loom-halt.md`.
````

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
