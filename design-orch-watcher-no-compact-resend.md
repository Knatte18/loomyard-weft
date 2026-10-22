# Orch watcher re-sends /compact after the compact already ran

`orch-watcher-no-compact-resend` · Note · bug, orch, creel

Imported from [issue #828](https://github.com/Knatte18/loomyard/issues/828).

Seen (loomyard hub orch, Neve, 2026-10-22): at a requested distill cycle the watcher posted inject #121 (/compact with the keep-argument) at 12:20:53Z. creel log: sending 12:21:08 (the compact ran; the session was compacted), stalled reason=draft 12:21:21, sending again 12:21:37, aged 12:22:00, stalled draft, sending again 12:22:49, aged 12:23:04. The operator saw /compact typed into the session twice more after the compaction and had to interrupt both. orch status meanwhile showed phase=resuming. watch.log also held at 12:23:05 'a creel call failed; the cycle is held until creel answers' because creel could not resolve the worktree of a run (report-timing-unreadable-journal) that had just been torn down after landing.

Workaround: the operator interrupts the repeated /compact by hand.

Expected: once a compact inject has been delivered and the session compacted, the watcher does not re-send it; a draft stall after delivery is not read as non-delivery. A creel strand lookup that meets a worktree removed mid-teardown skips it instead of failing the call.