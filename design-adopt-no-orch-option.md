# lym adopt always starts the orch, with no way to skip it

`adopt-no-orch-option` · Note · enhancement, orch, batten

## Seen

On 2026-11-09 the operator wanted to adopt root without starting the orch, to resume the orch session by hand with `claude -c`; adopt has no such option.

## Idea

An adopt flag that skips the orch start in its beam steps, leaving dobby to start as now.