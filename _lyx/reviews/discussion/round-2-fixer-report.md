# Round 2 fixer-report: discussion

## Changes

- F1 (BLOCKING), fixed as proposed: D9's "When the hold ends" bullet now decides that `stateChangeReady` measures its report wait from the hold's end, so a watched awaiting stop with a parent notice waits `noticeReportWait` from the hand-on for the stop report, then `noticeRelayWait` from it; a stop the hold never covered keeps today's rules.
  Premise confirmed in `internal/battenshed/notice.go`: `waited` is `now - episode.since >= noticeReportWait`, and `!qualifies` returns `waited`.
  The criterion is split: the hold-ending criterion drops the false relay-wait claim, and a new criterion pins the hand-on case the fix key describes.
  The rule is stated for every held stop, not only the awaiting one, since a held `blocked` stop's report also arrives only after the verdict.
- F2 (MEDIUM), fixed with the proposed mechanism's direction, decided concretely: `PrepareRunSettings` changes to validate-then-migrate for every module, so darn's `CheckValues` hook (D2's shared check) refuses `review_max_bounces: 0` as written and the migrated `skip_review` never reaches `ValidateLocal`.
  The fix key's example says only "before the Migrate hook runs"; running `ValidateLocal` as well as `CheckValues` before `Migrate` keeps one order for all modules.
  The review's concern about loom is addressed in D4: `loomengine.MigrateConfig` raises only `step_idle_timeout_min`, which no value check reads, so loom's silent repair is unchanged.
  The criterion now says the refusal never names `skip_review` as a key the task set.
  Scope's `configreg` item names the order change; support log Q13 records the pick.
- F3 (LOW), fixed as proposed: D9's end-condition bullet excludes a hand-over's stop report and park marker (a `hand-over` settle record at this stop key later than the stop); a criterion pins that a watched `blocked` eligible stop handed over to a fresh driver gets no notice before the successor's arbiter hands on.
- F4 (LOW), fixed as proposed, first option: D5's shared function answers the verb's whole `awaiting` and `blocked` eligibility (D5's cause rule and D6's verify-halt rule); D9's bullet names both rules.
- F5 (NIT), fixed as proposed: the Constraints line cites the `darn.yaml: review_max_bounces below 0` row by title.
- Support log: a round-2 entry under Review rounds.

## Disputed

None.

## Deferred

None.
