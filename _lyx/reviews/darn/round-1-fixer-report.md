# Fixer-report, round 1

## Changes

- F1 (fixed as proposed, commit `0a7cf9ab4`): `internal/shuttleengine/claudeengine/doc.go` now says the open launch carries `--dangerously-skip-permissions` unless the open's permission mode is `prompt`.
- F2 (fixed as proposed, commit `c4e82d618`): `internal/shuttlecli/cli_test.go` gains `TestOpenCmd_ListModelsRegistryLoadFailure`.
  It writes an unparsable `models.yaml`, runs `--list-models`, and asserts an error envelope whose message names `--list-models` and that no session opened.
  It is a sibling test, not a row, because the success test's assertions do not apply to an error envelope.

`lyx gate test ./internal/shuttlecli --tier 0 --tier 1` passed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
