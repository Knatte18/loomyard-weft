# Raddle checks refuse a part slug the board would refuse

The proposal checks now apply the board's slug rules to every filed slug, so an over-long slug fails before the operator approves.
Before, a proposal could pass `lym loom raddle-propose` and approval, and then stop at Raddle-File.
That happened to the part `single-repo-records-on-code-removal`, whose 35-character slug exceeds the board's 32-character limit.

## What changes

- `raddleparser.Check` reports a new finding, `slug-invalid`, for a filed slug, part or project, that the board's slug rules refuse.
  Its detail carries the refusal and the way forward: pick a slug the board admits, then run `lym loom raddle-propose` again.
- `BoardContext` gains a `ValidateSlug` function, which `Check` calls once per filed slug.
  A nil `ValidateSlug` checks nothing, so a caller that sets none sees no new finding.
- `raddleBoardContext` in `internal/loomcli` sets it to the board's length and `root` rules, then the pair slug rule with the hub's junction names.
- `boardengine.ValidateEntrySlug` is new and exported: the length limit and root reservation that `NewTask` applied inline.
  `NewTask` now calls it, so the board's write and the raddle check share one implementation.
- `contracts/specs/raddle-proposal-spec.md` lists `slug-invalid` and states that every filed slug is one the board's slug rules admit.

## Design decisions

- The check takes a function in `BoardContext`, not the board's limits, so `Check` stays pure and `raddleparser` does not import the board or fabric engines.
- The pair slug rule is included, not only length and `root`, because `lym board` writes and pair creation refuse a slug by it as well.
  A slug that passes length but fails the pair rule would otherwise stop at the same later step.
- Tests cover the check rows in `TestCheck`, the detail text in `TestCheck_DetailNamesTheWayForward`, and the real closure in `TestRaddleBoardContext`.
  The closure test configures `pathspec: docs` so that only the hub's junction names refuse `docs`.

## To check by hand

- Finding `junction-names-wiring-unpinned` (`internal/loomcli/raddlecontext.go`): confirm that `TestRaddleBoardContext` pins the hub's junction names in the `ValidateSlug` closure, through the `docs` row.
- Finding `kept-detail-test-lacks-keep-marker` (`internal/raddleparser/check_test.go`): confirm that `TestCheck_DetailNamesTheWayForward` carries its `testtiming:keep` marker, since `TestCheck` never asserts detail text.

## Merge danger

**Door:** two-way
**Blast radius:** raddle
