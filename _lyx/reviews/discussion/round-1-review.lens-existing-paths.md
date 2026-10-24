```yaml
lens: existing-paths
outcome: reported
```

Paths traced, per decision:

- D4 `ParentOf`: `boardcli` `cfg.ParentOf` (`internal/boardcli/cli.go:141`), `pairteardown` `HubSeamOptions.ParentOf`, adopt deps, batten owner, loom `ownercheck`, fabric remove `Effective` — all named as owner readers handed the one `ParentOf` — survives.
- D5 `Parent` vs adopt's `Add`: adopt step 2 calls `Add` from the prime with no `Parent`; an adopted weft keeps its existing origin record (`internal/fabricengine/add.go:157`, `:440`), so the first machine's recorded parent survives — survives.
- D6 default-branch origin: `add.go:441` writes `ParentWorktree: l.WorktreeName` when no record exists; D6 states the override for the default branch — survives.
- D8 guard: `shedcli` `RefuseSeedAt: battencli.RefuseUnlessPrime` (`internal/shedcli/table.go:62`) keeps prime-only, matching Scope Out — survives.
- D8 `--window` in the acting worktree's reed session vs `pairteardown`'s session end — breaks when the acting worktree is a task pair (F2).
- D10 weft reset vs `Add`'s keep of an adopted weft carrying a record (`add.go:157`): reset keeps the origin record — survives.
- D11 Worktree-Teardown of the beam vs `pairteardown`: ends registered processes and the reed session; the beam orch's watcher is a separate unregistered process (`internal/orchcli/spawn.go:30`) — breaks, record silent (F3).
- D12 `RegenerateHubWorkspace` callers: `regenerateAfterVerb` (`internal/fabricengine/hubworkspace.go:53`), `ideengine` spawn (`internal/ideengine/spawn.go:51`) — function itself changes, both survive.
- D13 `ResolveParent` callers: `hubgeom` strand/burler/webster parent names (`internal/hubgeom/hubgeom.go:83`, `:138`, `internal/hubgeom/webstergeom.go:45`) computed at launch — staleness covered by Open risk 2, survives; `loomcli` `newParentReviewConfig` (`internal/loomcli/wiring.go:390`) names the parent reviewer — live orch is the right reviewer, survives; `loomcli` decision `parent` seam (`internal/loomcli/decision.go:751`) — survives; `battencli` `noticesReachDriversParent` (`internal/battencli/wire.go:542`) — named in D13, survives; `loomcli` `parentRecordsCommitter` (`internal/loomcli/landingdeps.go:464`) — breaks (F1).

Findings:

**F1 — BLOCKING — design — D13: `ResolveParent` is also Finalize's parent-pair resolver.**
D13 redefines `hubgeom.ResolveParent` to return "the nearest worktree whose orch strand is live … the prime whatever its state when none is".
Finalize's `parentRecordsCommitter` uses `ResolveParent(l).Worktree` to locate the parent PAIR and commit its run records before the parent-side merge.
Under D13 a member of a beam with no live orch resolves to the prime, so Finalize commits the prime's run records and leaves the beam's (the real merge target) uncommitted; a live orch further up likewise misdirects.
The record never names this caller; the parent-pair lookup must stay on the origin record's `ParentWorktree`, separate from live-orch routing.
Evidence: `internal/loomcli/landingdeps.go:459-480`; `internal/hubgeom/parent.go:35-53` (Worktree comes straight from the origin today).

**F2 — MEDIUM — design — D8: a batten window opened from a task pair dies with that pair.**
D8 opens `lyx batten run --window` in "the acting worktree's reed session", and the widened guard admits any drivable worktree, task pairs included (`internal/fabricengine/fabric.go:162`).
Operator or agent in task pair A runs `lyx batten run B --window`: B's driver window lives in A's reed session; when A lands or is `cancel --remove`d, `pairteardown` ends A's reed session (`EndSession` session end, `internal/pairteardown/teardown.go:209-217`) and kills B's batten driver mid-run.
D8's self-pair refusal only covers the acting worktree being the pair acted on.
The record needs to bound which session hosts the window (e.g. beam's or prime's when the acting worktree is a task pair) or refuse task pairs.

**F3 — MEDIUM — design — D11: beam teardown leaves its orch watcher running.**
The beam's landing ends with Worktree-Teardown removing the beam pair; `pairteardown` ends registered processes (`runproc`) and the pair's reed session, which kills the orch strand, but the orch watcher is a separate `lyx orch watch` process (`internal/orchcli/spawn.go:30`) that nothing in `orchcli` registers with `runproc`.
It keeps running against `.lyx/orch/` paths in a removed worktree.
D8 puts the beam orch's watcher in the beam ("its watcher is its own"); D11 never says what happens to a beam's orch at landing — stop it first, refuse land while it is live, or register the watcher so teardown ends it.

Tool calls:
1. grep `internal/fabricengine/add.go` for origin handling — needed whether an adopted pair keeps an existing origin record (D5/D10 paths).
2. grep callers of `ResolveParent`/`parentNameOrEmpty` — the manifest had the definition, not every reader D13 changes.
3. sed contexts of four `ResolveParent` callers (`loomcli/wiring.go`, `loomcli/landingdeps.go`, `loomcli/decision.go`, `battencli/wire.go`) — to see what each uses the result for.
4. grep `runproc`/session end in `orchcli`, `orchengine`, `pairteardown` — whether teardown reaches the beam orch's processes.
5. grep how the orch watcher is spawned — to confirm it is a separate unregistered process.
