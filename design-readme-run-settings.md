# Show each task run_settings in the board README

`readme-run-settings` · Task · enhancement, board

## Seen

The board README (rendered by `internal/boardengine/render.go`) does not show a task's `run_settings`, so the review budget a task will run with, such as `{"darn":{"review_max_bounces":1}}` or the low budget `{"loom":{"review_max_bounces":2,"review_hard_max_bounces":3}}`, cannot be read there.

## Scope

- Show each task's `run_settings` in the README, compact, for example `darn: review_max_bounces 1`.
- Add no table column: the tables are already hard to read. Stack the settings vertically inside an existing cell instead, one setting per line, for example under the title.
- A task without `run_settings` renders as today.
- Update `internal/boardengine/doc.go` where it describes the README layout.

Operator decision 2026-10-17: no new README column; stack in height instead.
