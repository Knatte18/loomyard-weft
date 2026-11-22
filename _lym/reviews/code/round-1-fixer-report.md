---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
---

# Fixer-report, round 1

## Changes

- F1, F2, F3, F4 (`internal/fabriccli/roleguard.go`, `roleguard_test.go`): `inOwnSandboxRoot` now takes the parsed strand name.
  It resolves the target through its deepest existing ancestor and walks to the nearest `.lym/harden-sandbox`.
  A container that is a records sibling maps back to its owner worktree.
  The owner must resolve as a worktree of the strand's slug, in the hub whose recorded shortname is the strand's shortname.
  The owner's own sandbox root, resolved the same way, must be the root the walk found.
  The tests build the production layout: a worktree's `.lym` is a directory link into its `-weft` sibling.
  New rows cover the resolved `-weft` form, a link into and out of the sandbox, a same-slug worktree in another hub,
  a sandbox root under a subdirectory of the worktree, a relative `--into`, and `pull --weft-path` from the own sandbox hub.
- F3 follow-up, from the self-check: the owner's sandbox root goes through the same resolver, so a first clone before the root exists passes.
  A path on the way that exists but cannot be resolved, a dangling link, now refuses.
  Rows cover both.
  The `inOwnSandboxRoot` doc comment line that ran past 120 characters is split.
- F5 (`fabric.go`): the header states the sandbox-root exception.
- F6 (`doc.go`): the Merge line states the sandbox-root exception.

Gate repair: the comment lines the gate flagged in `fabric.go` and `roleguard.go` (the `fabric.go` header, `resolveThroughExistingAncestor`, `inOwnSandboxRoot`)
were rewritten as one sentence per line, with no line wrapped mid-sentence.

Commits: one for F1–F4 (shared files), one each for F5 and F6, and one for the F3 self-check follow-up, which changes a file the first commit already changed.
The `fabriccli` gate passes.

## Self-check

- fix-code-correctness, LOW, the over-long doc comment line: fixed.
- fix-code-correctness, LOW, a dangling link passes the resolver: fixed.
- fix-proposal, LOW, a first clone before the sandbox root exists is refused: fixed.
- fix-proposal, LOW, a hub with no recorded shortname refuses its own sandbox: disputed, premise false.
- fix-consistency: no findings.
- fix-affected-set: no findings.

## Disputed

- Self-check finding of fix-proposal, LOW, the shortname check.
  Ground: premise false.
  A strand name that parses carries a shortname, which the hub's record is the source of, so a hub with no record names no strand and the case cannot arise.
  The fix-consistency fork confirmed this against `internal/fabricengine/shortnamebinding.go`.
  Evidence: `internal/hubgeom/parent.go:41`.

## Deferred

None.

## Rulings

None.
