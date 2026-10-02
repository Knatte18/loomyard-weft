# Webster: verify gates Merriam, Merriam waits instead of polling, and agents are named for what they are

`webster-verify-gate` · Planned · feature

## Goal

A webster run's integration tests run once per attempt, as a mechanical gate on Merriam, and a failure goes back to Merriam to fix; Merriam waits on its forks instead of polling.
An operator reading `lyx reed list` sees each agent named for what it is, Merriam's strand included.

## Today

- Merriam's stencil (`contracts/stencils/webster/webster-template-master.md`) spawns an integration fork that runs the plan's `## verify:` once and writes `OK | FAILED`; on `FAILED` Merriam fixes nothing and stops.
- After Merriam exits, Go runs the same `## verify:` again, with no strand and an empty `activity.wait`, so the suite runs twice and the second run is invisible.
- `webster-recovery` (D5/D6) adds one fix attempt by a separate session at run exit, with the bisected card as a hint; on its first run the bisect named card 07 for a line card 09 wrote.
- Merriam polls `lyx webster await-batch` in the foreground every ~30s for as long as a fork runs, and polls the integration report with `sleep 20; test -f`.
  The stencil says a backgrounded wait ends the turn and kills the run; that predates shuttle's `EventWaiting` (`internal/shuttleengine/claudeengine/background.go`), which reads a turn end with a running background fork as still running.
- Merriam's strand shows as `<shortname>:<slug>:master`: webster spawns it with `string(RoleMaster)` (`internal/websterengine/runlevel.go`), the same value that keys its model in `webster.yaml`.
- Most spawning modules pass their role as an inline literal in their spawn spec.

## Fix direction

- Merriam's strand gets a shuttle gate (`shuttleengine.GateOutcome`, as Discussion-Write and Burler use) that runs the plan's `## verify:` when Merriam reports done.
  A failing gate injects its findings into Merriam's session; Merriam forks a fixer with the full plan context, and the gate re-runs, up to an attempt cap Discussion sets.
- The gate is the only run of the suite: Merriam's integration fork and Go's separate post-exit verify go.
- This replaces `webster-recovery`'s D5/D6 recovery session; Discussion decides whether its bisect survives as a hint in the gate's findings.
- While the gate runs, `activity.wait` names it.
- Merriam spawns each fork in the background, ends its turn, and acts on the fork's completion notification; the stencil's polling and its "started non-interactively" framing go.
  Discussion confirms Merriam's strand is waited on through the shuttle loop that reads `EventWaiting`.
- Discussion measures whether a re-run after a fix hits Go's test cache, and if the integration tests defeat it, re-runs the failed packages before the full suite.
- Merriam's strand gets the role `webster`, separate from its model key; whether the `master` model key in `webster.yaml` is renamed too is for Discussion, and a rename must keep existing configs loading.
- Each module keeps owning the roles it spawns, as a named constant beside its spawn site, not an inline literal.
  There is no central role list; `agentname`'s role set stays open, so a new module with its own agent touches no shared file.

## Done when

- A webster run runs the plan's `## verify:` once per gate attempt, visible in `activity`.
- A regression introduced by a card goes back to Merriam and is fixed in the same run, pinned by a test.
- Merriam's stencil contains no polling loop, and a batch completes with Merriam idle between the fork spawn and its notification.
- A webster run's Merriam strand is named `<shortname>:<slug>:webster`, and every spawn site takes its role from a named constant in its own module.
- The webster module doc and `docs/overview.md` describe the gate; the Agent Name Invariant in `CONSTRAINTS.md` states that the spawning module owns its role, and CLAUDE.md's Merriam terminology entry stays accurate.

## Depends on

`webster-recovery` (both edit `internal/websterengine` and Merriam's stencil) and `run-parentage` (both edit how agent names are formed and the Agent Name Invariant); start after both land.

