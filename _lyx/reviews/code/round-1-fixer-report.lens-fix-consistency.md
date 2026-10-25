```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix set for stale twins and sibling text through the consistency lens.

**Findings**

1. **LOW** — `docs/overview.md:630`
   - The text still says the batten overview "reads files only". The F2 fix changed the same claim in `internal/battencli/overview.go` (Long text), in `internal/battencli/cli.go` and in `dobbyreads.go`: the pane now also takes the dobby and driver locks for an instant.
   - The twin in the docs was left as it was.

2. **LOW** — `internal/loomcli/doc.go:178`
   - It says the loom driver "takes the run's driver lock without blocking … and a second driver refuses at once".
   - If the loom driver goes through `sheddriver.DriveVerb`, that is now false: `DriveVerb` waits `lock.ProbeGrace` (250ms) before it refuses.
   - I did not open `loomcli` to confirm it uses `DriveVerb`.
   - Check `internal/loomcli` for the call and for other `DriveVerb` docs, such as the `sheddriver` package doc and the `battencli` driver docs. If it does use `DriveVerb`, change the wording.

3. **LOW** — `internal/lock/lock.go`, `ProbeGrace` doc and its call sites
   - `ProbeGrace` is documented as the wait of an acquirer that "refuses a held lock". Only `DriveVerb` and `dobbyengine.Run` use it.
   - Other try-acquirers of the same driver lock were left as try-acquire. Examples: `battencli/arm.go:896` takes the run lock (not the driver lock, so possibly out of scope), and `battencli/land.go:118` takes the land lock.
   - I found no other driver-lock try-acquire path outside `DriveVerb`, but I did not check them all.
   - No action is needed unless one of those is the driver lock.

**Tool calls**
- 1 grep over `.go` and `.md` files (excluding tests) for "reads files only", "without blocking", "refuses at once" and the old prime-only wording. The context held no search of the repo for stale twins of the changed claims.
