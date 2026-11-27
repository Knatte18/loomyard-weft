```yaml
lens: fix-claims
outcome: reported
```

Task: check that the claims the fix added or rewrote hold against the code at HEAD.

**LOW: card 6, the new `--board`/`--landing` sentence.** "with either, it uses `--worktree` only to locate the hub and builds `BoardTarget` or `LandingTarget` over the prime."
- Today `pusherCmd` resolves every `--worktree` through `lymcwd.ResolveWorktree(worktree)` and then `runpusher.TargetOf` (internal/loomcli/pusher.go:43-55).
- `resolveCore` reads the hub as `filepath.Dir(workTreeRoot)` and then runs `anchorRelOrRoot` on the base name (internal/lymcwd/lymcwd.go:126-127). Nothing shows that this resolves for the Bolt/board directory, which is what `SpawnSync` passes (internal/heddlecli/cli.go:173 passes `c.boltDir`).
- The card doesn't say that the `--board` path skips `ResolveWorktree`/`TargetOf` and reads the hub from the directory's parent. An implementer who keeps today's resolution could still fail the detached child, which is the defect F8 was about.
- The new `TestPusherCmd_PushNowWakesAndExitsAtAHeldLock` row would catch this, so it is LOW and not MEDIUM.

The other new claims checked out:
- **Card 7 bound:** "the second process exits at the lock". `Run` returns nil when `deps.Lock()` is not acquired (internal/runpusher/run.go:95-101).
- **Card 17:** a `LockWait` of a few milliseconds fails at once. That holds, because only a zero value blocks (`LockWaitUnbounded = 0`, internal/fabriclayout/pushtypes.go:111-112). The card must not let the test leave it zero, and as written it doesn't.
- **Card 20:** `hubDobbyDepsStarting` (beamcli) can fill `Waits` from `battencli.HubRunSlugs`. beamcli already imports battencli (internal/beamcli/deps.go) and battencli doesn't import beamcli, so there is no cycle. `HubRunSlugs` covers the prime's runs and each local beam's (internal/battencli/catchup.go:321-323).
- **Card 9:** the removal's job is handed to the teardown-lock holder after the local steps (internal/fabricengine/remove.go:345-358). `archiveWeftTip` pushes only when `job.Kind == remoteTeardownRemove` (internal/fabricengine/remoteteardown.go:123). The archive tag takes `tip` separately from the push (internal/fabricengine/archive.go:145), so tagging the whole tip while pushing only up to `<hold>^` is possible.
- **Card 11 step 2:** the rewritten premise matches `HandOverOwnedElsewhere` passing `RunPush{}` and having no reset of its own (internal/pairteardown/handover.go:220).

Tool calls:
- Read `runpusher/run.go`'s lock, `fabriclayout`'s `LockWait` constants and the `ResolveWorktree` doc: my context lacked the pusher verb's lock behaviour and the meaning of a zero wait.
- Grep of `resolveCore` and the `SpawnSync` callers: to see what directory `--worktree` would receive for the board and how it resolves.
- Grep of the beamcli/battencli imports: to check card 20's new `Uses` for an import cycle.
- Read `pusherCmd`'s `RunE` and the lock grep: to see how `--worktree` resolves today.
