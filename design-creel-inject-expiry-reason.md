# creel inject into a bash strand expires with no stall reason, and a stalled inject expired 3 s late

`creel-inject-expiry-reason` · Note · bug, creel

Imported from [issue #686](https://github.com/Knatte18/loomyard/issues/686).

Seen: `inject lyt:dave ... --ttl 10s --wait` into a bash strand ended `expired` with no `stall` field, while Claude-adapter panes now name theirs. Separately, a 20 s TTL into a too-short pane expired at 23.3 s, while a never-idle pane 30 s TTL expired on time at 31.4 s.

Expected: an expired inject names a reason, such as "not an agent pane", or the outcome is documented; a stalled inject expires at its TTL.

Found by the creel stage 1 re-test of 2026-10-16.