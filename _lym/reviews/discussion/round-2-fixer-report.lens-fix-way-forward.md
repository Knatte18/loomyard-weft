```yaml
lens: fix-way-forward
outcome: reported
```

The fix's ways forward hold in most places, with three gaps:

1. **MEDIUM** — Decision 5, the `dobby.out` rotation and its bound: "The process that takes the lock … rotates `dobby.out` … and reopens its stdout and stderr onto the fresh file" and "A rotation whose rename fails, as on Windows when another process holds the file open, logs its error … and leaves the file growing until a later start rotates it."
   - **Summary:** on Windows the lock holder renames a file it holds open itself, so every start's rotation fails there. The stated recovery, a later start, never comes, and `dobby.out` has no bound on Windows.
   - **Evidence:** `SpawnDetached` hands `dobby.out` to the child as its fd 1 and 2 (`internal/dobbyengine/start.go:276-277`, which the fix now points at `dobby.out`). The reopen that would release the file runs only "Where `selfexec` execs", and `internal/selfexec/identity_windows.go:3` says Windows has no exec. The holder therefore always holds the file it tries to rename. The bound names "another process" holding it, but the blocker is the holder itself.

2. **MEDIUM** — Decision 1, the worktree-gone exit: "stops a board-tidy agent in flight through `TidyShuttle.StopStrand` … and logs the stop, or a failed stop with its error".
   - **Summary:** the stop runs through a shuttle runner that is built lazily over the removed worktree's location. A re-exec'd image that has not built that runner yet tries to build it from config dirs that are gone, so the stop can fail in exactly the case it exists for.
   - **Evidence:** `internal/beamcli/dobbydeps.go:169` wires `&runnerTidyShuttle{worktree: worktree}`. `dobbydeps.go:421` builds the runner on first use through `battencli.ShuttleRunner(s.worktree)`, and `internal/battencli/cancelcmd.go:419-437` loads reed and shuttle config from `fabricengine.ConfigDirs(location)` of that worktree. Even with a runner, `shuttleengine/stop.go:58-60` cannot find the run under the removed anchor and only removes the strand.
   - **Effect:** when the build fails, the strand stays open and the prime log line names no step for the operator. The record says neither that the runner must be built over the prime's location nor what the operator does with a failed stop.

3. **MEDIUM** — the new Acceptance criterion "A dobby started with its stdout and stderr on `dobby.log` writes its later envelope and stderr lines to `dobby.out` after a start."
   - **Summary:** this test cannot run as the record's tiers stand.
   - **Evidence:** the reopen of fd 1 and 2 happens only "Where `selfexec` execs", in `runDobby`. Inside a `go test` process it would redirect the test binary's own output, and `SpawnDetached` returns `ErrSpawnUnderTest` under `go test` (`start.go:261-262`). So the criterion needs a real detached process, meaning tier1 with the prebuilt lym. The Constraints still say only the binary-replacing test is tier1 and "everything else is tier0 with injected clocks". The criterion carries no tier mark and the Constraints name no fd-redirect seam.

The other ways forward are reachable and make progress: the refused re-exec's second read, the posts for a fatal `Run` error and a post-resolve `placeDobby` refusal, the one-time emptying of `LandedSinceTidy`, and the `lym fabric reconcile` then `lym gate test --gate adopt` warning (`lym fabric reconcile` exists, `internal/fabriccli/fabric.go:316`).

Tool calls:
- Grepped `internal/fabriccli` for the reconcile verb, to confirm the warning's `lym fabric reconcile` step exists; the parent's context held only `gatecli`'s text naming it.
- Grepped the `StopStrand` wiring and `shuttleengine` stop, to see whether the worktree-gone stop can reach a strand; the parent read only the `TidyShuttle` interface.
- Read where `runnerTidyShuttle` is built and the start of `ShuttleRunner`, to learn which worktree the runner is built over; the context lacked it.
- Read `hubDobbyDeps`' notice wiring and `primeShuttleRunner`, to see what config the lazy runner loads; the context lacked it.
