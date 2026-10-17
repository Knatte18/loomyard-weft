# Fixer report — plan, round 1

## Changes

- F1 (BLOCKING, scope): fixed as proposed, taking the review's first option.
  In `_lyx/plan/01-log-id-reminded.md`, `internal/creelcli#TestCreelCLI` joins the Edit list after `internal/creelengine_test#TestScheduler_Reminders`, and `internal/testkit/creelfake#DormantStrand` joins Uses.
  After the checked-assertions sentence, the Intent now names the `log --id prints a carried entry's kind` subtest in `internal/creelcli/cli_test.go`, explains why D4 makes its one-entry assertion fail or flake, and prescribes registering `beta` as `creelfake.DormantStrand(beta)`.
  That option is deterministic, because `worker.locate` reports a strand that is not live as not running (`internal/creelengine/deliver.go:236`), so `probe` returns before `remind` and journals no reminder.
  The ImpactSummary now says one existing creelcli assertion changes.
- F2 (MEDIUM, design): fixed as proposed.
  In `_lyx/plan/00-overview.md`, the untagged verify line is now `lyx gate test ./internal/creelengine ./internal/creelcli ./tools/sandbox`.
  The integration line is unchanged, because `tools/sandbox`'s tests carry no build tag.
- F3 (NIT, consistency): fixed as proposed.
  In `_lyx/plan/02-refusal-prefix-once.md`'s first test row, the refusal is now quoted as an unknown receiver (`creel: unknown receiver "..."; the names that exist: ...`), matching `internal/creelengine/pool.go:204`.

## Self-check

- fix-consistency: no findings.
- fix-claims: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings; it left no finding unchecked.

## Disputed

None.

## Deferred

None.
