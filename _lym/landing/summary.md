# lym retro: run the retro task's writer on Opus

The retro task's writer judges transcripts, separates real faults from noise and routes each proposal.
On the `docs` profile it ran on Sonnet, and a weaker writer files plausible proposals that become board noise.

`Plan.TaskFields` in `internal/retroengine/retro.go` now adds `run_settings` `{"darn":{"writer":"opus"}}`,
so every retro task's darn writer runs on Opus.
The model-spec is the named constant `TaskWriter`.
`TestPlan_TaskFieldsFilesOneDarnTaskOnTheDocsProfileWithAnOpusWriter` pins the writer in the rendered fields.
The retro entry in `docs/overview.md` states the writer setting.

## Design decisions

- The writer is set in the task's `run_settings`, which the board carries per task, so the `docs` profile and the hub's darn config stay unchanged.
