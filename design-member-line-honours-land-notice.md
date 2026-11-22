# A beam member gets its landed line only when its land_notice is on

`member-line-honours-land-notice` · Note · bug, dobby

## Seen

`per-task-settings-one-lookup` (`118bee8ad`) landed with an exception in `rememberRuns` at `internal/dobbyengine/landnotice.go`:
while a beam's pull request is recorded, a member gets its landed line whatever its `land_notice` reads.
The line carries the follow-up asking the orch to update the change description and run `lym beam describe`.
Its Code-Burler raised this as F12, `member-line-overrides-explicit-off`; the run landed before anyone ruled on it (see `held-decision-lands-unruled`).

## Decision

Operator, 2026-11-22: no exception.
Per-task beam settings follow one rule, and `land_notice` off means no line.

## Change

- `rememberRuns` sends a member's line only when its reading is on; the follow-up stays on that line.
- The package doc at the top of `landnotice.go`, the doc comments of `landNotices` and `rememberRuns`, and `LandNoticeState.Member` drop the exception; `Member` goes if nothing else reads it.
- `PATTERN-landing.md` and `pattern/PATTERN-config-layers.md` lose any mention of it.
- Keeping the pull request's description current after each member landing without the orch is its own note, `beam-pr-describes-itself`.
