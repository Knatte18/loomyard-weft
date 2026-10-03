# Verify localizes failures, gates Merriam once, and Publish never runs on a dirty tree

`verify-publish-hardening` · Planned · feature

Depends on: [`test-suite-derived-guards`](design-test-suite-derived-guards.md)

## Goal

A failing integration test stops on the card that caused it, Publish never verifies or pushes a dirty tree, and no gate is skipped after a crash.
refusal-audit stopped five times on integration-verify regressions that the plan-level verify could not localize.
Webster's plan verify runs once, as a gate on Merriam that sends a failure back to Merriam to fix, and Merriam waits on its forks instead of polling (`webster-verify-gate` below).

## Also folded in, not in bolt before

- refusal-audit self-report 2: card `verify:` commands run package-scoped, so `cmd/lyx` repo-wide invariant scans first run at the plan level; Plan-Write could append `go test ./cmd/lyx/`.
- refusal-audit self-report 3: Webster's integration verify runs against the working tree, so an uncommitted edit can mask a non-compiling HEAD.
- Publish's conflict resolver reuses `.lyx/loom/conflict-resolution-r1.md` across Publishes (`internal/mergeresolve/spec.go:60`), so a later conflicting merge-in is refused while an earlier file exists; refusal-audit hit it.
- Deliver every agent prompt through a file, so no role can hit the 30000-byte launch limit (`internal/shuttleengine/claudeengine/command.go`); Master already does (`e39909bbc`).

## From `card-verify-integration`: Cards that touch integration-tagged tests verify with -tags integration

No refusal-audit card declared a verify: line; the fork gate is go build plus the package's unit tests (webster-body-implementer.md:30), untagged, while 11 of 48 touched test files are //go:build integration.
Want: Plan-Write adds `go test -tags integration <pkg>` to a card's verify when it touches a tagged file; a message-rewording card greps the repo for the old text first so stale exact-string assertions become part of the card; Master re-runs integration tests after a rework fork.

## From `publish-dirty-tree`: Publish must not verify or push a dirty tree after merge resolution

refusal-audit's Publish merge left historyfold_test.go uncommitted and verify ran on the dirty tree; earlier publish-review-fixback had the same uncommitted merge-resolution edits

## From `verify-gate-crash-after-merge`: Landing verify gate skips a merge interrupted before its marker write

Found reviewing PR #308 (`publish-verify-after-merge`).
Fabric reports a finished-but-interrupted merge as already up to date, so the gate cannot tell a verified tree from an unverified one.
Robust fix: record the task HEAD that last passed verify, and verify whenever HEAD differs from it.

## From `webster-verify-gate`: verify is a gate on Merriam, Merriam waits instead of polling, and its strand is named webster

A webster run's integration tests run once per attempt, as a mechanical gate on Merriam, and a failure goes back to Merriam to fix; Merriam waits on its forks instead of polling.
Merriam's strand is named for what it is.

### Today

- Merriam's stencil (`contracts/stencils/webster/webster-template-master.md`) spawns an integration fork that runs the plan's `## verify:` once and writes `OK | FAILED`; on `FAILED` Merriam fixes nothing and stops.
- After Merriam exits, Go runs the same `## verify:` again, with no strand and an empty `activity.wait`, so the suite runs twice and the second run is invisible.
- `webster-recovery` (D5/D6, landed `3c791470d`) added one fix attempt by a separate session at run exit, with the bisected card as a hint; on its first run the bisect named card 07 for a line card 09 wrote.
- Merriam polls `lyx webster await-batch` in the foreground every ~30s for as long as a fork runs, and polls the integration report with `sleep 20; test -f`.
  The stencil says a backgrounded wait ends the turn and kills the run; that predates shuttle's `EventWaiting` (`internal/shuttleengine/claudeengine/background.go`), which reads a turn end with a running background fork as still running.
- Merriam's strand shows as `<shortname>:<slug>:master`: webster spawns it with `string(RoleMaster)` (`internal/websterengine/runlevel.go`), the same value that keys its model in `webster.yaml`.
### Fix direction

- Merriam's strand gets a shuttle gate (`shuttleengine.GateOutcome`, as Discussion-Write and Burler use) that runs the plan's `## verify:` when Merriam reports done.
  A failing gate injects its findings into Merriam's session; Merriam forks a fixer with the full plan context, and the gate re-runs, up to an attempt cap Discussion sets.
- The gate is the only run of the suite: Merriam's integration fork and Go's separate post-exit verify go.
- This replaces `webster-recovery`'s D5/D6 recovery session; Discussion finds why the bisect named the wrong card and decides whether it survives as a hint in the gate's findings.
- While the gate runs, `activity.wait` names it.
- Merriam spawns each fork in the background, ends its turn, and acts on the fork's completion notification; the stencil's polling and its "started non-interactively" framing go.
  Discussion confirms Merriam's strand is waited on through the shuttle loop that reads `EventWaiting`.
- Discussion measures whether a re-run after a fix hits Go's test cache, and if the integration tests defeat it, re-runs the failed packages before the full suite.
- Merriam's strand gets the role `webster`, separate from its model key; whether the `master` model key in `webster.yaml` is renamed too is for Discussion, and a rename must keep existing configs loading.
### Done when

- A webster run runs the plan's `## verify:` once per gate attempt, visible in `activity`.
- A regression introduced by a card goes back to Merriam and is fixed in the same run, pinned by a test.
- Merriam's stencil contains no polling loop, and a batch completes with Merriam idle between the fork spawn and its notification.
- A webster run's Merriam strand is named `<shortname>:<slug>:webster`.
- The webster module doc and `docs/overview.md` describe the gate, and CLAUDE.md's Merriam terminology entry stays accurate.


## Drop begin-batch's model assertion (#345, operator 2026-10-03)

`begin-batch` asserts Merriam's model before every fork; the fork inherits Merriam's model, so the assertion has nothing to check and goes.
