# Split round 2 — fixer report

Both findings and every gate finding fixed in `_lym/raddle/proposal.yaml`.

## Changes

- **F1 (BLOCKING), fixed as proposed.**
  Three whole criterion bullets moved out of `transition-push-takeover`, verbatim:
  - "D1: a run claim, a landing's board `done`, an owner write and a hold write are on origin when their verb returns." to `machine-registry-holds-catchup`;
  - "D2: an adopter's takeover push rejected over the old owner's later transition…" to `adopt-foreground`;
  - "D2: after an in-loop hand-over no local pair of the slug stands…" to `leftover-pairs-teardown`.

  Each target part has `transition-push-takeover` on its `depends_on` path, so the carried D1 and D2 ids stay built.
  `transition-push-takeover`'s `delivers` also names the hand-over's local pair removal through `pairteardown`'s local mode when the caller runs outside the pair, as D2 has it (`_lym/discussion/decision-record.md:95`).
- **F2 (MEDIUM), fixed as proposed, by moving the reader to a package both can import.**
  - `machine-registry-holds-catchup`:
    - `takes` now lists `readTries` and `writeTries` in `internal/adoptcli/holds.go` with their callers, `fabricengine.AdoptScratchDir` and dobbyengine's import allowlist (`internal/dobbyengine/seam_enforcement_test.go`).
    - `delivers` places the upgrade's first read at the dobby start, before its catch-up, in `internal/dobbyengine`.
      It moves the tries file's reader and writer into a new `internal/fixtries` as `fixtries.Read` and `fixtries.Write`, importing only `internal/state` and taking the subject's directory from the caller.
      This keeps dobbyengine's told-geometry allowlist rule and leaves no import cycle, since `internal/adoptcli` imports `internal/dobbyengine`.
      `delivers` also switches adoptcli's callers to the new functions, adds `internal/fixtries` to the allowlist and has the first read call `fixtries.Read` on each subject's directory under the adopt scratch directory its caller hands in.
    - `exports` gains `fixtries.Read` and `fixtries.Write` with their production callers.
    - `packages` gains `internal/fixtries` at size 40, and `cost_line` now reads about 2410 changed lines, matching the summed package sizes.
  - `adopt-foreground`'s `takes` and `delivers` now name `fixtries.Read` and `fixtries.Write` in `internal/fixtries` in place of the reader in `internal/adoptcli/holds.go`.
- **Gate, `criterion-uncarried` and `criterion-unknown`.**
  An earlier pass split the D1 and in-loop hand-over bullets instead of moving them whole; the gate accepts only verbatim Acceptance criteria bullets.
  The split bullets are gone, and both original bullets sit whole in their F1 target parts above.
- **Gate, `board-unchecked`.**
  `board_checked` gains the open entry `claude-code-mode` at its `revised` time `2026-11-16T10:15:37Z`, read from `lym board list`.

`jq` parses the proposal, each moved bullet sits in exactly one part, and each part's summed package sizes match its cost line.

## Disputed

None.

## Deferred

None.

## Rulings

None.
