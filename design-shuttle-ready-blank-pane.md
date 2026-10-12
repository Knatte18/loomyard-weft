# Shuttle marks a working agent died when its pane is collapsed

`shuttle-ready-blank-pane` · Note · bug, shuttle

Shuttle's startup probe decides readiness only from a pane capture (`internal/shuttleengine/wait.go`, through `claudeengine/startup.go`), looking for the prompt glyph or `shortcuts` on screen.

On 2026-10-12 convergence-plan's parent-review alignment agent (run dir `4e8c64f5…`) started, took its prompt and was mid-review when shuttle marked it died at `startup_timeout_s` (90 s) and removed its strand.
`startup-capture.txt` held a single newline: the pane was blank.
The likely cause is reed's layout: loom status, driver, Discussion-Write and the alignment strand exceed the 44-row window, so `clampToFit` (`reedengine/render/height.go`) floors a pane to about one row.
The review fell back to the orch, so the verdict path held but ran without the alignment pre-pass.
It was the first use of the alignment agent after `alignment-agent` landed.

## What to do

- Treat a recorded `SessionStart` plus a `UserPromptSubmit` or tool event in the run's `events.jsonl` as proof of readiness, ahead of the capture.
- Give the probe a minimum pane height, or capture scrollback, so a collapsed strand cannot read as never-ready.
