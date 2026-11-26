```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at HEAD.

**MEDIUM** — card 8, the RewriteRunSettings paragraph: "snapshots `board.json` and the rendered board files before its `boardCriticalSection` call, and writes them back itself when `beforeCommit` … or any later step returns an error; a failed write thus leaves the board's bytes as they were."
Summary: as written, the restore runs after `boardCriticalSection` has returned, which means it runs outside the board's write lock, so the guarantee does not hold under concurrent writers.
Evidence: `internal/boardengine/board.go:103-107` takes the lock with `defer lock.Release()`, so the lock is gone once the function returns at `:187-189` or `:194-196`. Between that release and `RewriteRunSettings` writing its snapshot back, another board writer (`lym board upsert`, a status set) can take the lock and save. The restore would then overwrite that write with the stale snapshot. The snapshot is also taken before the lock is acquired, so it may not match what `boardCriticalSection` loaded and saved. To hold, the snapshot and restore have to happen under the lock: take the snapshot inside `fn`, and restore inside the `afterSave` wrapper before it returns the error. A render failure at `:194` still falls after `afterSave`, and nothing inside the lock covers it.

**LOW** — card 8: "A commit or push failure in the detached sync never reaches the walk; the board's own sync retries it."
Summary: the first half holds, but nothing I read shows that the board sync retries a failed commit or push.
Evidence: `internal/boardengine/spawn.go:39-64` starts one detached `lym board sync --deferrable`, logs a spawn failure and returns. It has no retry, and the boardengine doc says nothing about retrying. If it holds, it holds only because the next board write's sync commits the tree. The claim should say that, or say less.

Checked and holding:
- Card 9: the shed pre-run skips `shed` and `seed`, and every recipe arm (`loomcli.ArmAt`, `battencli.ArmAt`) runs after `lymcwd.Resolve` in it (`internal/shedcli/cli.go:166`, `:180-189`).
- Card 9: `loomCLI.arm`'s start-and-resume call is at `internal/loomcli/arm.go:221-225`, and `battenCLI.arm`'s run-only call is at `internal/battencli/arm.go:300-304`.
- Card 9: the pusher starts in the Bolt directory (`pusher.go:24`). Its pre-push check, `newPrePushCheck` (`prepush.go:52-80`), shows no loom or tapestry load, and `runReconciler` reads neither config, so "load neither config" holds for the pusher. I did not read the checklist verb's body far enough to confirm it loads neither config.
- Card 5: `configreg.PrepareRunSettings(settings map[string]any)` matches the call in the new row (`internal/configreg/runsettings.go:43`).
- Card 2: "line 7" and "lines 77-78" match `pattern/PATTERN-config-layers.md`.

Tool calls:
1. Grepped for `newPrePushCheck`, `checklistCmd`, the `PrepareRunSettings` signature and `spawnSync`: none of these were in context, and the pusher, Seed-Child row and sync claims depend on them.
2. Grepped the pre-push check file for config loads, which the first grep's pattern missed.
3. Read `newPrePushCheck`'s body and grepped for sync retry, to check "load neither config" and "retries it".
4. Read the `newPrePushCheck` span and the boardengine doc's sync text, which were not in context.
5. Grepped `runReconciler` for config loads and `boardcli`'s deferrable sync, to finish the pusher and retry checks.
