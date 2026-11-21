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
---

## Changes

- F1: added the `//testtiming:keep` line above `TestCheck_DetailNamesTheWayForward` in `internal/raddleparser/check_test.go`.
- F2: `TestRaddleBoardContext` seeds the hub's fabric config with `pathspec: docs` through `hubforge.SeedFabricConfig` and adds a row wanting `ValidateSlug("docs")` refused.
- F3: reworded the `ValidateSlug` paragraph in `internal/loomcli/raddlecontext.go` and the `slug-invalid` bullet in `contracts/specs/raddle-proposal-spec.md`.
  The self-check's consistency lens found the twin doc on `BoardContext.ValidateSlug` in `internal/raddleparser/check.go`, which got the same rewording.
- F4: the spec's filed-slug rule sentence now also says the board's slug rules admit the slug.

Gate: `lym gate test` tier 0 for `./internal/loomcli` and `./internal/raddleparser` passed, and tier 1 for `./internal/loomcli` passed with `TestRaddleBoardContext` run.

Gate repair: the `ValidateSlug` doc in `internal/raddleparser/check.go` broke a sentence across two lines.
It is now two sentences, one per line, each under 120 characters; no test ran before the repair.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, `BoardContext.ValidateSlug` doc at `check.go:98` left unreached by F3; fixed in `a42f5b8e`.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
