# Card 2 — weft-branch-resolver-checkout

**Create:**
- `plan:internal/fabricengine#resolveWeftBranch` -> `func resolveWeftBranch(rec *Mutations, l *lyxcwd.Location, weftBranch string, consultOrigin bool) (exists bool, fromOrigin bool, err error)`
- `internal/fabricengine/weftbranch.go`
- `internal/fabriccli/adoptremote_integration_test.go`

**Edit:**
- `internal/fabricengine#Topology.switchOrForkWeft`
- `internal/fabricengine#Topology.Checkout`
- `internal/fabricengine/checkout_rollback_test.go`

**Uses:**
- `internal/fabricengine#weftBranchExists`
- `internal/fabricengine#remoteHeadTip`
- `internal/fabricengine#WeftRepoRoot`
- `internal/fabricengine#KindBranchCreated`
- `internal/fabricengine#Topology.rollbackSwitch`

**Intent:**
D5's one weft-branch resolver, landed with its first caller, `checkout` (D7's checkout half).

`resolveWeftBranch` lives in the new file `internal/fabricengine/weftbranch.go`, with a file header naming D5's order:
1. local `refs/heads/<weftBranch>` exists (`weftBranchExists`): return `exists=true, fromOrigin=false`;
2. else, when `consultOrigin` is true and the weft repo has an `origin` remote: probe `origin` with `remoteHeadTip` in the weft repo root; when the branch is there, fetch it into its remote-tracking ref (`git fetch --no-tags origin refs/heads/<b>:refs/remotes/origin/<b>`) and create the local branch tracking it (`git branch --track <b> origin/<b>`), record `KindBranchCreated` via `rec.AppendRef` with `refDetail("weft", weftRepoRoot, "origin")`, and return `exists=true, fromOrigin=true`;
3. else return `exists=false`, and the caller forks as today.
A weft repo with no `origin` remote (detected the way `cleanup.go` does, `gitrepo.New(weftRepoRoot).RemoteURL(originRemoteName)` failing) skips step 2.
A failed probe, fetch or branch creation is a returned error naming `origin` and the branch, never a fall-through to step 3.
All git runs through `gitexec.Run` (`PATTERN-fabric-git`, `PATTERN-gitexec-checked-call`); no new `//gitexec:raw` site.
Bound: step 2 only creates a local branch whose content is already on origin; it pushes, replaces and deletes nothing, and the new failure mode is an unreachable origin, which now stops the caller instead of forking.

`switchOrForkWeft` calls `resolveWeftBranch(rec, l, weftBranch, true)` in place of its `weftBranchExists` check.
An existing branch (local or just created from origin) is switched to, recording `KindWorktreeSwitched` as today; the method reports `forked=true` when the branch was created from origin as well as when it forked, so `Checkout` hands it to `rollbackSwitch` as a created branch and a later failure deletes only that local branch through the existing `deleteBranch` gate, leaving origin untouched.
When the weft switch itself fails after the resolver created the branch from origin, `switchOrForkWeft` returns `created=true` together with the error, and `Checkout` hands that branch to `rollbackSwitch` on the error path too, so the branch is deleted through `deleteBranch` and no origin-created branch outlives a failed checkout.
Rename the return to `created` and update its doc comment and `Checkout`'s.
A resolver error returns from `switchOrForkWeft`, and `Checkout` rolls the warp side back through its existing `rollbackSwitch(..., "")` call, so both sides stay on their original branches and no weft branch is forked.
Checkout adopts an origin weft whether or not an archive tag covers it.
Bound (D7): checkout creates no pair and only switches an existing weft worktree onto a local branch tracking content origin already holds; the operator named the branch, nothing on origin changes, and D4's notice reports config changes at landing.
A hub whose weft repo has no `origin` remote forks as today.

Tests:
- `internal/fabriccli/adoptremote_integration_test.go` (new, `integration` tag, package `fabriccli_test`, `hubforge` hub, every verb through `fabriccli.RunCLIIn`): this card adds the checkout rows: in a pair, a warp branch `<b>` exists locally and only `origin/<b>-weft` exists (pushed from a scratch clone of `h.WeftBare`, carrying a marker file) — `lyx fabric checkout <b>` switches the weft worktree to a local `<b>-weft` that tracks `origin/<b>-weft` and holds the marker, also when an `archive/<b>/*` tag covers that origin tip; with the weft repo's `origin` URL pointed at a missing path and no local `<b>-weft`, the verb exits non-zero naming `origin`, both worktrees stay on their original branches, and no `<b>-weft` exists locally; with the weft repo's `origin` remote removed, checkout forks as today.
- `internal/fabricengine/checkout_rollback_test.go`: a row in the junction-failure rollback test where the weft branch came from origin: after the injected failure the local `<b>-weft` is gone and the bare's `<b>-weft` tip is unchanged; and a second row where the weft switch itself fails (an untracked file in the weft worktree at the path of the marker file the origin branch tracks, which the pre-switch tracked-dirty check ignores and `git switch` refuses to overwrite): the local `<b>-weft` created from origin is gone, origin is unchanged and both sides stay on their original branches.

**ImpactSummary:** fabricengine Checkout's weft branch selection now consults origin before forking; rollback scope unchanged apart from origin-created branches counting as created.

**Commit:** 2: fabric: resolve a pair's weft branch from origin before forking; checkout adopts it
