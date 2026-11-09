# A green adopt check starts adopted runs the operator paused

`adopt-check-overrides-pause` · Note · bug, tender, dobby, batten

## Seen

On 2026-11-09, after `lym adopt root`, the orch set `lym batten pause` and `lym tender pause` on the three adopted runs before the adopt check ended.
When the check went green, its holds lifted and dobby started all three runs anyway; the orch paused them again within a minute.
The same start happened on 2026-11-02 with no operator go-ahead.

## Idea

Lifting the adopt hold must never override a pause set by anyone else.
Adopted runs should stay paused after a green check until the operator resumes them.