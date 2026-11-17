# lym retro: run the retro task's writer on Opus

`retro-writer-opus` · Task · enhancement, report, darn · running · Drift-Baseline

## Seen (2026-11-17)

`lym retro` files its task as darn on the `docs` profile with no writer model (`TaskFields` in `internal/retroengine/retro.go`),
so the writer runs on the docs profile's default model, Sonnet.
The scripts under `tools/retro` do the counting; the report's worth lies in judgement:
telling a real fault in a transcript from noise,
routing each proposal to a check, a rubric line, a deletion or a prompt sentence,
and ranking them by severity.
A weak writer files plausible proposals that turn into board noise.

## Idea

- `TaskFields` adds `run_settings` `{"darn":{"writer":"opus"}}`, so every retro task's writer runs on Opus.
- Its test pins the writer in the rendered fields.
