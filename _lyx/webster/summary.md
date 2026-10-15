# shed-tackler: the Go driver and its tackler as a shed layer

All four batches reached `done`; `record-batch` returned no warnings for any of them.
The scope guard listed changed files outside each batch's own targets as informational deviations; they are the call sites the moves retargeted.

## Batch 01 — interrupt-policy-words (cards 1–3), head `932c294f4`

The interrupt-policy words moved from loomshed into shedverbs.
`internal/sheddriver` opened with the stop key, settle records and tackler records.
The settle records moved to a per-run `shedrun` path, and every reader was pointed at it.
Deviations: loomcli, loomengine, arbiterstop, sheddriver and shedverbs files, the retargeted readers and moved files.

## Batch 04 — tackler-call-to-sheddriver (cards 4–6), head `87006b9dc`

The tackler call, its shuttle seam and its stencil (now `shed-template-tackler.md`) moved to `sheddriver`, recipe-neutral.
Loom's drive verb, envelope path and refusal stayed in `drivewiring.go`.
The settle loop, rule table and hand-back moved into `sheddriver` behind optional seams.
Deviations: loomcli files whose logic moved.

## Batch 07 — shed-config-module (cards 7–12), head `63d7c20db`

The `shed` config module landed with its defaults and pure resolvers.
Loom, darn and batten keys became overrides of `shed.yaml`, and a hub's `loom.yaml` defaults and comments migrate once.
`PATTERN-shed-driver` and its seam test were added, with the told-geometry binding.
Batten's until-stop loop and interrupt policy are armed, and a batten stop episode sends one orch notice shared with the row notices.

## Batch 13 — batten-drives-its-run (cards 13–14), head `dbd4d179f`

`lyx batten run` drives its own run through the `sheddriver` body in-process.
The overview and package docs follow the move.

## Verify gate fixes

The first gate run failed `TestCLIForm_EveryCommand` in `cmd/lyx`: `lyx batten run`'s `Long` text was 2057 bytes against the 2000-byte cap.
The fixer trimmed the Go-driver paragraph, the Seed-IDE/Open-IDE sentence and the `--window` paragraph in `internal/battencli/cli.go`, dropping the sentence on a second batten being refused by the run's own lock.
Fix commit: `adc608bca` (`fix: trim lyx batten run Long under the 2000-byte cap`).
The fixer ran `lyx gate test --tags integration ./cmd/lyx ./internal/battencli`, which passed.
