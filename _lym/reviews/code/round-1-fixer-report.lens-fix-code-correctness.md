```yaml
lens: fix-code-correctness
outcome: reported
```

Lens: fix-code-correctness. I ran the changed code and added tests step by step, and found two LOW issues and nothing worse.

**LOW — `endArchiveEpisode` success-path message**
- Changed text: `internal/dobbyengine/handover.go`, the `d.endArchiveEpisode()` call that follows a successful `ArchiveOwner` in `handOver`.
- Problem: `Recover` logs "the beam needs no hand-over any more; the archive failure ends". On this path the hand-over is happening and the dobby exits right after, so the message is false. It is also redundant, because the exit line follows at once.
- Evidence: `endArchiveEpisode` is called after the archive succeeded and before the "another machine owns this beam; the dobby exits" log.
- Suggested change: pass a neutral message, or reset only `archiveFailing` on that path.

**LOW — an unreadable owner ends the episode**
- Changed text: `probeOwner`, the `!elsewhere || holds` branch that now calls `endArchiveEpisode`.
- Problem: `beamOwnedElsewhere` returns `false` when the owner read fails, so a read error also ends the failure episode. If the beam is still foreign when the read works again, the next archive failure counts as a new episode and logs again.
- Evidence: the `ownerRead.Fail` path in `beamOwnedElsewhere` (`loop.go`) returns `false, ""`.
- Impact: this only costs an extra log line, which is benign.

**Checks that held**
- **F1:** the `listed` check in `ownerSubject` returns the slug itself.
  - **Fix:** `Covers` now reads true for `free` and for `nest` under the listed `free`.
  - **Rows:** before the fix, both walked up to `root`, which the record does not list, so both rows fail without the fix.
  - **Unchanged:** the rows for `deep` and `torn` keep their old results.
- **F5:** the new test fails without the fix. The `archiveFailing` flag never reset, so the second episode wrote one line and the count would be 1, not 2.
- **F4:** the new test fails without the fix.
  - **Landing step:** `landBeam` calls `CanLand` first when `Slug` is set, so the `landReads` spy catches the landing gate.
  - **Other steps:** `drivePasses`, the task start, tidy and test-map are gated by `scheduling`, and each has its own spy.
  - **Tidy and test-map timing:** a landing is first counted at tick 2, when `perTick(2)` runs inside the tick hook, so those steps would already have fired by then if ungated.
  - **After the lift:** the probe after the lift hands over, so the run stops at two ticks and `len(h.ticks) == 2` holds.
- **F7:** the done-run row and the recorded-elsewhere row both fail without the skip, since `r1` would get paused. The failing-listing row and the failing-Bolt-pull test fail if the owner read or archive were skipped.
- **F2, F8:**
  - **Ahead row:** it expects no tags. A broken ancestor check would tag the tip and the row would fail.
  - **Diverged and absent-branch rows:** they expect a tag.
  - **Origin branch assertion:** it compares the origin tip before and after, so a branch push would be caught.
  - **Torn-pair test:** the detached `b` fails on its code side while the `root` tag is still produced.
- **F3, F6:** the pair-set test pins exactly the tags for `r` and `root`, or for `b`, with `b` unpushed but excluded for the prime. The skip test fails without the skip, because the tag would exist after the first call.
- **Callers of the changed `ownerSubject` signature:** the only caller is `covers`.

**Tool calls (1 of 6)**
1. `grep` of `landBeam` in `internal/dobbyengine/land.go`: my context lacked its early-return conditions, which I needed to know whether the `CanLand` spy detects a missing gate.
