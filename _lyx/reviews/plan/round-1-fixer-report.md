# Round 1 fixer-report — plan

## Changes

- **F1** (fixed as proposed): `_lyx/plan/02-shell-guard-table.md` Intent gains a sentence after the Move group sentence.
  Each shell row's comment on its `standingDenies` entry moves unchanged onto that row's entry in `Rows`, and only the sentence saying that ending with `; true` guarantees exit 0 goes onto `renderShellGuard`.
  Premise checked against `internal/shuttleengine/claudeengine/settings.go`: the guardrail comments sit on the `standingDeny` entries.
- **F2** (fixed as proposed): `TestCheckTexts` in card 2 now plants a single quote, a double quote and a backslash in the steer, and those three plus a line break in the notice.
  This matches `steerTextForbiddenChars` and `noticeTextForbiddenChars` in `settings.go`.
- **F3** (fixed as proposed): card 6's `loadConfigOrRefuse` wrap reads `claudeengine: %w; way forward: correct or remove the key in the file the error names, with lyx config claude --layer <default|worktree|local> (or --set for a value)`.
  The `engineWithInvalidConfig` rows of `TestPrepare_RejectedBeforeArtifacts`, `TestOpenLaunch` and `TestPromptCacheLifetime` now also assert the refusal names `--layer`.
  Premise checked against `internal/configcli/doc.go`: without `--layer`, a verb addresses the module's own file.
- **F4** (fixed as proposed): card 5's `Uses` section, holding only `internal/configreg#checkLoad`, is deleted.

`lyx loom validate-plan` reports `ok: true` with informational findings only.

## Disputed

None.

## Deferred

None.
