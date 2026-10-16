# creel inject --wait prints a draft stall from 37 s earlier as if current

`creel-stale-stall-line` · Note · bug, creel

Imported from [issue #682](https://github.com/Knatte18/loomyard/issues/682).

Seen: the owner `inject --wait` into lyt:bob printed `lyt:bob is stalled: the input box holds a draft ... (at 10:39:55)` 37 s after the draft had been cleared, then injected at once.

Expected: no stall line once the input box is empty, or one clearly marked as past.

Found by the creel stage 1 re-test of 2026-10-16 (sandbox scenario C8).