# Decision record — gate-test-runtime-caller-paths

## Goal

A test author who runs `lyx gate test` learns from the gate's own output that the tier builds with `-trimpath`, and so that `runtime.Caller` reports module-relative file paths, before debugging a path failure.

## Scope

- `internal/gatecli`: before a tier's commands run, print one stderr line naming the tier's effective GOFLAGS when it carries `-trimpath`, and write the same line into the tier's run log.
- The guard step goes through the same tier run, so it prints the line too.
- `internal/gatecli/doc.go` documents the line and why `-trimpath` stays.
- Out of scope: removing `-trimpath` from any GOFLAGS, the hub's `gate.yaml`, and the non-tier slot holders' `gateslot.HolderEnv`.

## Decisions

### Keep `-trimpath`, surface it

Decision: the gate keeps `-trimpath` in every tier's GOFLAGS where the hub's `gate.yaml` sets it, and the non-tier holders keep setting it; the gate names it instead of dropping it.
Rationale: the operator decision on the board entry, since `-trimpath` lets a fresh worktree reuse the build and test cache.

### The effective GOFLAGS decides

Decision: the line reads the last `GOFLAGS` entry of the environment the tier's commands get, the gate's inherited environment plus the tier's own `env`, and fires when a field of it is `-trimpath`, `--trimpath` or either with `=true`.
Rationale: the child process sees the last entry, so a GOFLAGS inherited from an outer gate run counts as well as one the tier sets, and a `-trimpath=false` field does not fire.

### Stderr and the run log, one line per tier run

Decision: the line goes to stderr and into the tier's run log, once per tier run that has an invocation, and names the tier, the GOFLAGS value, the effect on `runtime.Caller` and the way forward of locating the repository from the working directory or `go.mod`.
Rationale: stderr reaches the agent at the moment of a failing run, and the log keeps it beside the failure for anyone reading the record later; a tier with an empty scope runs no build and needs no line.

## Constraints

- `PATTERN-gate-slot` and `PATTERN-trimpath-safe-root`: every gate build keeps `-trimpath`.
- `PATTERN-comment-line-breaks`: one sentence per line in Go comments.
- `PATTERN-test-economy`: the line's presence extends the existing scenario subtest whose tier sets `-trimpath`; detection variants are rows of one table.

## Auto-mode assumptions

- A line on every trimpath tier run is acceptable noise, since the gate already prints a summary line per tier.
- The run record's JSON gains no field: the log carries the line.

## Open risks

- A `-trimpath` set on a tier's command line rather than in GOFLAGS gets no line; the command shows under `-vv` and in the run record.

## Acceptance criteria

- A tier whose env sets `GOFLAGS` with `-trimpath` prints the line on stderr naming the tier and its GOFLAGS, and its run log holds the same line.
- The detection fires for `-trimpath`, `--trimpath` and `-trimpath=true`, not for `-trimpath=false` or a GOFLAGS without it, and reads the last GOFLAGS entry of the environment.
- `internal/gatecli/doc.go` documents the line.
