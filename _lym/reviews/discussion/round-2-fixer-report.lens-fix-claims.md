```yaml
lens: fix-claims
outcome: reported
```

Task: check that each claim the F8 fix added or rewrote in the decision record holds against the code at HEAD.

**LOW: "the call costs one stamp read" is not accurate.**
- Changed text: D5, "once the stamp names the running build, the call costs one stamp read".
- Evidence: in `internal/hubreconcile/walk.go:44`, `ensure` reads the stamp, then runs `repoDefaultAbsent(geom.BoardDir)`. That function (`walk.go:90-99`) stats each per-worktree module's board config file.
- Before any of that, the verb resolves geometry through `hubgeom.ReconcileGeometry`, which calls `preflight.BoardLymPresent` (`internal/hubgeom/hubgeom.go:22-27`).
- A current stamp therefore costs one stamp read plus a stat per per-worktree module plus the board-presence check, not one stamp read.

**LOW: "the first verb of any kind on a new build runs the walk" overstates the sentence's own rule.**
- Changed text: D5, "so the first verb of any kind on a new build runs the walk".
- Evidence: the same sentence makes the rule apply only to "every lym verb that loads the loom or tapestry config". A verb that loads neither config, such as a board-only verb, does not run the walk. The claim should read "the first such verb".

These claims hold:
- **`EnsureOrWarn` failure or lock timeout:** a lock timeout goes through `ensure` as `*LockTimeoutError`; `EnsureOrWarn` logs it and returns (`hubreconcile.go:67-71`). The load then sees the unmoved file.
- **Mid-merge worktrees:** the walk skips one and leaves the stamp stale (`walk.go:323-326`, `81-83`), so the next walk retries.
- **Incomplete pairs:** the walk passes one over without marking the walk skipped (`walk.go:306-309`), matching the first part of the second bound.
- **Local files:** the walk reaches the local files Seed-Child wrote. `ReconcileAll` reconciles each module's local file through `reconcileSparseFile` (`configsync.go:235`), so the "every file layer" rationale covers local files.
- **Plan-writer note:** "`loomcli`'s `arm`, which runs it today for `start` and `resume` alone" matches `internal/loomcli/arm.go`.
- **D7:** "the next walk retries" agrees with the stale-stamp behaviour above.

Tool calls:
- I read `configsync.ReconcileAll` and `hubgeom.ReconcileGeometry`. My context lacked them, and I needed them to confirm that local files are walked and what the per-verb call costs.
