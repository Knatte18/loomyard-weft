```yaml
lens: fix-proposal
outcome: reported
```

The fix-proposal lens found 2 findings.

**Findings:**

1. **MEDIUM.** D5's new paragraph says batten's done-motif-run clear "admits a start whose recipe needs a stored record … That start is judged as fresh". The clear it reuses leaves the motif run's task branch behind, so the "fresh" jacquard start can still be refused when it creates its pair, and the F3 criterion "after a done motif run … set to jacquard starts" can fail.
   - `ClearMotifRun` calls `clearDoneRun(prime, slug, fabricCleanupDeps(prime), admitsMotifRun, nil)` with a nil branch discarder (`internal/battencli/cleanup.go:142`).
   - `ClearDoneRun` is the call that archives and deletes the task branch (`cleanup.go:145-167`).
   - `doneSlugRefusal` says a torn-down pair "keeps its task branch locally and on the remote", which "leads straight into the create row's leftover-branch refusal on the re-run" (`internal/battencli/arm.go:937-949`).
   - A jacquard run makes its own worktree and branch under that slug.
   - Unverified: whether a raddle start ever meets a leftover branch under the same slug. If it does, some create-row handling I did not read may already cover this.
   - The fix should say the clear also discards the motif run's task branch for a jacquard start (the `ClearDoneRun` path), or name that refusal and its way forward.

2. **LOW.** D5's `lym loom start` bullet covers "a start made in a worktree directly". Its way forward is D4's jacquard sequence, which opens with `lym batten cancel <slug> --remove` and ends with `lym batten run <slug>`. Both steps assume a batten run owns the slug.
   - For a worktree whose jacquard seed was written by hand (`lym shed seed self --recipe jacquard`, which `disagreeingSeedRefusal` names as a seeding path at `internal/loomcli/sharedbootstrap.go:262-263`), no batten run exists to cancel.
   - So at the exact site the bullet scopes, the way forward may not be reachable.
   - Unverified: how `lym batten cancel` behaves on a slug with no batten run.

**Proposals checked with no finding:**
- **F7:** reconcile order is fine. `applyReinterpretations` runs after `ReconcileHubWideAt` has applied the template follow (`internal/hubreconcile/walk.go:250-266`), so the F4 reinterpretation sees the gained `defaults.jacquard`.
- **F4:** registering it in `internal/configreg` beside the arras one matches the registration at `configreg.go:171`.
- **F5, F6, F10, F12, F2:** I built no breaking case.

**Further tool calls:**
1. Grepped for where reinterpretations and `FollowTemplate` are called, to learn reconcile order for F4. Context held only the arras reinterpretation's body.
2. Read `walk.go:250-300` and `hubreconcile/doc.go` for the order of follow and reinterpretation. Context lacked the walk.
3. Grepped for `startClearsMotifRun` callers and read `refuseAtStart`, for the F3 proposal's reach. Context held only lines 160-200.
4. Read `startguard.go:120-160` and the `ClearsMotifRun` consumers, to see which clear the F3 start triggers. Context lacked the guard wiring.
5. Read `ClearMotifRun` and `ClearDoneRun` in `cleanup.go`, to see what the clear removes. Context lacked that file.
