# Fixer-report — code, round 1

## Changes

- **F1** (fixed as proposed): `internal/heddleengine/store.go` refusals now say `--targets` (`missingFlags` and the no-target message in `ValidateEntry`).
  `setCmd` in `internal/heddlecli/verbs.go` passes a non-nil empty slice when `--targets` was given empty, so `ValidateEntry` refuses it and the entry is unchanged.
  Tests: the `NewWithoutTargets` row now expects `--targets`; a `Set` row `ExistingWithEmptyTargets`; a tier-1 CLI case `EmptyTargetsOnExistingEntryIsRefused` (added after the self-check showed the CLI branch was untested).
- **F2** (fixed as proposed): `Decode` now returns a blob-fitting refusal, "fix the file with a new commit on the branch that holds this blob"; `Load` keeps the `git checkout HEAD` advice. `decodeFrom` became `decodeBytes`, shared by both.
  Test: `MalformedBlobFailsDecodeWithBlobWayForward`.
- **F3** (fixed as proposed, partly): `docs/overview.md` lines 172, 173, 375 and 376 use `<BoltDir>` and `fabricengine.BoltDir`; lines 119, 143, 182 and 370 say "the Bolt checkout".
  Line 126 keeps the `_board` literal on purpose: it names the reserved slug token, and `lymcwd.BoltDirName` is still `"_board"`.
  Line 172's token list keeps the literal as "`_board` (the Bolt checkout)".

Commits: `aee75848` (F3), `9c0a2fa1` (F1 and F2 together, since both touch `store.go` and its test), then one commit for the self-check changes.

## Self-check

- fix-code-correctness, LOW: no test for the CLI branch of F1 — fixed (tier-1 case added).
- fix-consistency, LOW: `docs/overview.md` 375 and 376 still `<BoardDir>` — fixed.
- fix-consistency, LOW: line 126 kept the literal though the review listed it — kept on purpose, stated above.
- fix-consistency, LOW: line 172 less concrete than its sibling tokens — fixed, same change as the proposal fork's second finding.
- fix-proposal, LOW: F2 advice wrong for base and parent blobs — fixed with a side-neutral way forward.
- fix-proposal, LOW: line 172 reserved-token literal — fixed.
- fix-proposal, LOW: lines 173, 375, 376 stale `BoardDir` — fixed.
- fix-affected-set: no findings.

Lens reports: `round-1-fixer-report.lens-<lens>.md` beside this file, one per fork; all four returned a report.

## Disputed

None.

## Deferred

None.

Verification: `lym gate test ./internal/heddleengine ./internal/heddlecli` (tier 0) and `lym gate test --tier 1 ./internal/heddlecli` pass.
