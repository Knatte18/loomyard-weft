---
format: 5
approved: false
language: go
---

# Plan: verify-publish-hardening

A failing test stops on the card that caused it, the plan-level verify runs once per attempt as a must-pass gate on Merriam that sends a failure back to Merriam to fix, Publish never verifies or pushes a dirty tree, no verify is skipped after a crash, and no agent prompt can hit the launch-line limit.
The plan builds one shared verify function with a per-worktree verified-tree record first, moves every verify site onto it (landing, the Webster-Burler gate, the new webster gate, the self-check verb), then retires webster's post-exit integration stage, bisect, triage baseline and integration-fix strand.

## Card Index

1 — verifytree-package — the shared verify function, verified-tree record and running marker
2 — gated-wait-defers-waiting — a gated shuttle run's waiting turn end is not an arrival
3 — pointer-launch — claudeengine launches every session with a pointer to prompt.md
4 — merge-stage-tracked — fabric verbs to stage tracked edits and list untracked files mid-merge
5 — mergeresolve-session-edits — the conflict session's tracked edits land in the merge commit; untracked files halt; stale reports cleared
6 — landing-clean-verify — Publish and Finalize verify through verifytree and halt on a dirty tree
7 — webster-geometry-verifydir — webster Geometry carries the verify directory
8 — webster-burler-verify-gate — Webster-Burler carries a must-pass verify gate
9 — status-verify-note — loom status names a running verify
10 — recover-batch-notification-wait — recover-batch blocks to terminal; Merriam waits only on notifications
11 — merriam-strand-role — Merriam's strand role is webster
12 — per-card-gate-command — begin-batch renders each card's gate command
13 — plan-write-reworded-messages — Plan-Write greps for reworded messages
14 — webster-verify-gate — the plan-level verify becomes a gate on Merriam's strand with a fixer fork
15 — retire-integration-stage — delete the integration stage, bisect, triage baseline and fix strand
16 — webster-verify-verb — lyx webster verify and the Verified-Tree Invariant
17 — docs-and-specs — specs, overview and CLAUDE.md describe the result

## Shared Decisions

### verifytree is the one plan-verify function

`internal/verifytree` (card 1) owns the clean-tree check, the record, the running marker and the run.
Its API, used by cards 6, 8, 9, 14 and 16:
`Dir(anchorRoot)` (`<anchorRoot>/.lyx/verify`), `NewPaths(worktree, dir) Paths`, `Site{Label, Attempt}`, `Verify(ctx, Paths, Site, command) (Result, error)` with `StatusPassed`/`StatusSkipped`/`StatusFailed`/`StatusDirty`, `DirtyPaths(worktree)`, and `ReadMarker(path)` with `Marker.Note()`.
Only `Verify` writes the record, and only after a pass;
no agent prompt names the record path.

### One verify directory per worktree

Every hub-mode site gets the same directory, `verifytree.Dir(<anchor path>)`:
landing through `landingshed.Deps.VerifyDir` (card 6), the Webster-Burler gate through `shedrecipe.Env.VerifyDir` (card 8), webster's gate and verb through `websterengine.Geometry.VerifyDir` (card 7), and loom status through the same call (card 9).
Standalone webster uses `verifytree.Dir(stateDir)`.
Each site keeps the worktree it already runs from.

### Site labels

`webster gate`, `Webster-Burler gate`, `Publish`, `Finalize` and `webster verify`, rendered by the status note as `verify <site> (attempt N, since HH:MM)`.

### Gate attempts

Both the webster `verify` gate (`verify_gate_attempts`, template default 3) and Webster-Burler's `verify` gate (`attempts: 3` in the recipe) are must-pass.
Attempt counts live in memory per shuttle run and restart from zero after an attach, bounded by the run's timeout.

### No migration

A webster run in flight with an `IntegrationFix` state record or an `integration.yaml` report restarts with `lyx webster run --fresh`;
`state.json` decoding tolerates the retired field.
A hub `webster.yaml` still carrying `poll_wait_s` keeps loading.

## verify:

```
go build ./...
go test ./...
go test -tags integration ./...
go vet -tags smoke ./...
```
