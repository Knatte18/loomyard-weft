# Raddle recipe run: stuck at batch 07-report-config

Two of eight batches are recorded `done`; the run stopped at batch 07.

## Batch 01-shuttle-usage-split (done, head `5a209ac6`)

Cards 1–4: `SessionUsage` carries the four token counts apart, the Codex engine reads a session's usage from the rollout, shuttle calls a runner hook once per ended session, and shed has one usage writer with the `_lym/usage/` record files.
The fork reported a `SessionID` field on the session record that the card's field list omitted, and a finalize test that uses the timeout outcome as its non-done row because the repo has no stuck outcome.
The digest listed deviations in `internal/shedadapters`, `internal/shedrun`, `internal/shuttleengine` and `internal/shuttleengine/codexengine`; no departures or renames.

## Batch 05-usage-wiring (done, head `b3944e81`)

Cards 5–6: every run runner records its sessions, status commits carry the usage directory, Seed records recipe, profile and stored-record flag, and Finalize records the landed diff size.
The digest listed deviations in `internal/fabricengine`, `internal/landingshed`, `internal/loomcli`, `internal/shedadapters` and `internal/testkit/envkit`; no departures or renames.

## Batch 07-report-config (failed, recovery refused)

The fork committed cards 7–12 (head `d7f0465e`) and reported all gates green.
It built the report config module in `internal/reportconfig` instead of the planned `internal/reportengine`, because `configreg` importing `reportengine` closes an import cycle.
`record-batch 07` then failed its done-check: the Create targets `internal/reportengine/template.yaml` and `reportengine#Config`, `#ConfigTemplate`, `#ConfigSpec`, `#LoadConfig` and `#TestLoadConfig` do not resolve.
My parent recorded a decision (mail #706) moving card 7's targets to `internal/reportconfig` and card 8's config load to `reportconfig#LoadConfig`, but the plan files still name `reportengine`.
`recover-batch 07` refused with `batch_failed` on the same unresolved targets, so the ladder ends the run here.
The plan needs card 7 and card 8 matched to `internal/reportconfig` before the run resumes.

## Not run

13-board-promote-raddle, 19-raddle-checks-and-state, 23-raddle-recipe, 24-batten-raddle-start-guard and 26-report-raddle were not begun.
