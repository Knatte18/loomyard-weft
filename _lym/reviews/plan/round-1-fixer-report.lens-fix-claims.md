```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: I checked each claim the fix added or rewrote against the code at this worktree's HEAD. All but one hold.

1. LOW. Card 9, `AwaitFirstRead`: "a gone relay's recorded refusal is an earlier relay's".
   - **Problem:** this overstates the case. The relay this `away` just started can record a retryable refusal, such as an unknown pane process or a daemon that is down, and then die of another cause. A gone relay's refusal can therefore be the current relay's, and `away` then names the log at the bound instead of that refusal.
   - **Evidence:** card 9 step 4 says any other refusal "writes `FirstReadRefusal`, is retried …", so a relay that crashes after that write is gone with its own refusal recorded.
   - **Fix:** the behaviour is still sound. Narrow the rationale to "a gone relay's recorded refusal is never the strand-pane refusal of a relay this start launched".

These claims hold:
- `resolveSender`'s doc comment says "no other refusal does" (`internal/creelengine/identity.go:46`).
- Off Linux, a claim returns through `resolveFromEnvironment` before any pane walk (`identity.go:52-54`).
- dobby's `CheckBinary` is `func(beforeExec func(), failed func(error))` (`internal/dobbyengine/loop.go:64`), wired with `selfexec.New().CheckWith` (`internal/beamcli/dobbydeps.go:119`).
- `Client.Call` delegates to `CallWatching`, which arms one timer (`client.go:137-146`).
- `internal/state` offers `UpdateJSON(path, lockPath, mutate)` (`state.go:159`), so the locked read-modify-write the card describes exists.
- `lymcwd.Resolve` applies the cwd gate and refuses a directory outside a git repo (`lymcwd.go:81`).
- creel's daemon verb resolves no location (`internal/creelcli/cli.go:873-905`).
- `fabricengine.PrimeName(l)` lists worktrees at `l.AnchorPath()` (`internal/fabricengine/worktreelist.go:150`).
- `lymcwd.NamedWorktree(hubPath, name)` exists, spawns nothing and needs no gate.
- `hubAnchorDeps` already resolves `primeName` (`internal/fabriccli/anchordeps.go:75`).
- `lock.AcquireWriteLockWithin(lockPath, wait)` and `lock.ProbeGrace` exist (`internal/lock/lock.go:23,61`).
- The existing failing-handler subtest is at `internal/creelcli/cli_test.go:1572`.

Calls made:
- Read `identity.go:40-60`, `lymcwd.Resolve`, dobby's `CheckBinary`, the `internal/state` function list, the creel daemon verb and `Client.Call`. The context held only grep fragments of these, not the claims' targets.
- Read the exported `internal/state` functions and `lymcwd.Resolve`'s doc. The first listing was cut off before `UpdateJSON`, and the gate's behaviour wasn't in the context.
