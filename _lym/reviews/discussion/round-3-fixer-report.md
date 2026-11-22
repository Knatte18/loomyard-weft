---
dispositions:
  - id: F12
    disposition: fixed
---

# Round 3 fixer-report

## Changes

- F12 (fixed): D4 step 2 of `_lym/discussion/decision-record.md` now says `lym beam ready` commits the ready record in the beam's run-records worktree with no push (`MarkReady` in `internal/beamcli/deps.go`, through `fabricengine.CommitAnchoredPaths`), so it sits on `<beam branch>-weft` and never on the beam branch.
  The local read in the beam's worktree stays.
  The remote read is `git -C <records repo> fetch origin`, then `git -C <records repo> show origin/<beam branch>-weft:<anchor>_lym/landing/beam-ready.json`.
  The `<anchor>` prefix is added because `CommitAnchoredPaths` scopes the path under `AnchorRel`.
  A line notes that the records store's refspec maps a one-repository hub's `refs/lym/heads/` to `origin/`, so the read works in both layouts.
  For a beam on another machine, an absent file proves nothing, so the orch asks that beam's orch or the operator, and the beam qualifies only on a no.
  The D4 acceptance bullet now says "on the beam's records branch".
  `_lym/discussion/support-log.md` gains a Round 3 entry for F12 that replaces Round 2's read on the beam branch, and Round 2's F11 line stays as written.
- Self-check fixes in `_lym/discussion/decision-record.md`:
  - D1's verb list names `lym creel post` to another beam's orch.
  - D4 step 2 counts a record as the flag only when its `slug` is the beam's (`ReadyFor` in `internal/pawlengine/ready.go`).
  - A missing `origin/<beam branch>-weft` counts the same as an absent file.
  - The ask to the beam's orch falls back to the operator after ten minutes with no reply.
  - The D4 acceptance bullet covers the ask.

## Self-check

- fix-consistency, LOW, D1's verb list omits creel to another beam's orch: fixed.
- fix-consistency, LOW, D4 acceptance bullet omits the ask for a beam on another machine: fixed.
- fix-claims: no findings.
- fix-proposal, LOW, any `beam-ready.json` is read as the flag although lym also checks its slug: fixed.
- fix-way-forward, LOW, the ask to the beam's orch has no bound: fixed with a ten-minute fallback to the operator.
- fix-way-forward, LOW, a missing `origin/<beam branch>-weft` is not named as an outcome: fixed.

## Disputed

## Deferred

## Rulings
