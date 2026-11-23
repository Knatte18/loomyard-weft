```yaml
lens: fix-proposal
outcome: reported
```

Findings, newest-risk first.

**MEDIUM — F3 fix (`sweep.go` sweepSubjects and releaseGoneHolder, `chase.go` isOpenQuestion): it settles questions from other hubs' strands as `asker-gone`.**
- `isOpenQuestion(msg) && isStrandName(msg.From)` puts `msg.From` into the sweep subjects with no check of `msg.SendingHub` or `msg.SendingMachine`. `msg.SendingHub` is non-empty for inbound messages from other hubs (`fold.go:71-72`, `receive.go:318`).
- An inbound question keeps the other hub's strand name in `From`, and that name is a valid strand name.
- `sweepOnce` builds `present` from this hub's strand listing only, so that sender never appears in it. After `goneAfterSweeps` (2) misses, `releaseGoneHolder(from)` settles the question unanswered.
- Case: a strand of hub B asks a strand of hub A with `--expect-reply`. About 40 s later (30 s backoff plus the 10 s confirming sweep) hub A settles the question `asker-gone`, so the chase never runs, although the asker is alive on hub B.
- The proposal's premise, that an absent name means a removed strand, only holds for local senders. The new subject rule and the settle loop both need to exclude messages where `SendingHub != ""` or `SendingMachine != ""`.
- No test row covers this.

**LOW — F8: two production changes exist only for tests.**
- `Runner.HasReplyProbe` is a new exported accessor, and `burlerCLI.runner` is a field that exists only so a test can read it.
- `Runner.PauseGeometry` is a precedent, and the `export_test.go` route the review proposed cannot cross packages, so the deviation is justified.
- The `burlerCLI.runner` comment says it is kept for a test; that matches the rubric's "stale in a way a reader would notice" only if the field is later left unused.
- I'd keep it as it is.

**No findings** for F1, F4, F5, F6, F7 and F9. Reasons:
- **F1:** a stale non-orch `LYM_STRAND_NAME` in a detached process falls back to the program claim when the daemon answers `ErrStrandClaimOutsidePane`, so it stays correct.
- **F4:** the `"GATE"` program name is a plain upper-case name like the sibling program constants. The extra probe on the stop-only rig is harmless.

Tool calls:
1. `grep -n "SendingHub" internal/creelengine/*.go`: to confirm that inbound cross-hub messages keep a foreign strand name in `From`. My context lacked the `Message.SendingHub` semantics.
