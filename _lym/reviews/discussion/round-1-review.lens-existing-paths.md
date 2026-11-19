```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the existing code paths that write or read the state D1–D4 introduce or change.

**Paths traced, per decision**

- D1/D2, plain `gitrepo.Repo.Fetch` (`internal/gitrepo/pull.go:35`, a bare `git fetch` that obeys the configured refspec), reached from `handover.go`, `codetipsettle.go`, `pull.go:186`, `mergeguards.go:59`, `pushclassified.go`, `bolt.go:313` and `takeoverwrite.go`: **survives**. The mapped fetch refspec lands namespaced heads at `refs/remotes/origin/*`.
- D2, the tracking-ref move in `recordPush` (`internal/gitrepo/pushdestination.go:107`): **survives**. `refs/remotes/origin/<b>` matches the fetch mapping.
- D2, the upstream write in `recordPush` (`internal/gitrepo/pushdestination.go:114`): **breaks**. See finding 2.
- D2, explicit `refs/heads/` fetch sources: `add.go:520,621`, `checkout.go:215`, `weftbranch.go:49`, `remoteleftover.go:142,172`, `ownerarchive.go:317`, `remove.go:683`, `remoteteardown.go:557`. They break untranslated, but D2's "fetch sources" bullet covers them. No finding.
- D2, `ls-remote --heads/--tags` sites: `originreconcile.go:526`, `pushclassified.go:646`, `archive.go:318`, `remoteleftover.go:236,262`, `cleanupremotewarp.go:278,294`. D2's selection and read-back bullets cover them. No finding.
- D2, the remote `HEAD` read in `remoteDefaultBranch` (`cleanupremotewarp.go:87,309`, which parses `ref: refs/heads/`): **breaks**. D2 says it answers the board branch but gives no name to answer with. See finding 3.
- D2, `warpDefaultBranch` (`addadopt.go:61`): **survives**. It runs in the code worktree.
- D2, the way-forward text in `remoteteardown.go:260,273` (`git push origin refs/tags/<tag>` in the records store): **survives**. The configured push mapping applies to a remote-name push, per D1's evidence.
- D1, `reconcileWarpBinding` (`reconcile.go:340`), which backfills `.lym-warp` from the warp origin: **survives**. It writes the same URL D1 binds.
- D1/D3, CloneHub step 3b and the board worktree (`clone.go:304-326`), which read the weft primary's checked-out branch from a full clone's `HEAD`: **breaks** under D3's init-then-fetch store. See finding 3.
- D4, teardown's remote delete through `gitrepo.Repo.DeleteRemoteBranch` (`push.go:308-322`), with the destruction gate's records in `destroy.go:1631,1643`: **breaks**. See finding 1.
- `refscanner.go:557`, the `ls-remote` form: unaffected. It audits agents' commands, not fabric's own git calls.

**Findings**

1. **MEDIUM**, class `design`, location D4 in `_lym/discussion/decision-record.md`: deleting by the full namespaced name turns an absent remote ref into a reported deletion. The record says nothing about this.
   - `gitrepo.Repo.DeleteRemoteBranch` deletes by the short name on purpose. Its comment says "a full refs/heads/ name deletes a missing ref silently" (`internal/gitrepo/push.go:313`).
   - It returns `(false, nil)` for an absent ref (`push.go:319`), and the destruction gate uses that `deleted` flag to tell a removal from an absent ref (`destroy.go:1631,1643`).
   - The handler ran git 2.53.0 against a bare origin: `git push <url> --delete refs/lym/heads/missing-weft` printed `[deleted]` and exited 0. A short name can't reach `refs/lym/heads/` at all; deleting an existing `x-weft` by short name answered "remote ref does not exist".
   - So D4 forces the full name, and with it every never-pushed or already-removed records branch is recorded as deleted.
   - D4 needs to say how an absent namespaced ref is told apart, for example an `ls-remote` check first or the existing leased delete (`DeleteRemoteBranchLeased`, `push.go:327`).
2. **MEDIUM**, class `design`, location D2: the upstream config is written by `gitrepo`, not fabric, and D2 gives `gitrepo` the namespace for "push, lease and push-result matching" only.
   - `gitrepo`'s `recordPush` sets `branch.<b>.merge` to `"refs/heads/" + dest.branch` after every push that asks for an upstream (`internal/gitrepo/pushdestination.go:114`).
   - D2 calls this "the upstream config fabric writes" and leaves it out of `gitrepo`'s parameter list. Built as worded, the records store keeps `refs/heads/<b>` as its merge ref, which the fetch refspec `+refs/lym/heads/*:refs/remotes/origin/*` can't map.
   - The D2 criterion "a records branch's upstream … names its namespaced ref, and the branch reads as in sync" then fails. The in-sync read at `recordsreads.go:175` reads that key.
3. **MEDIUM**, class `design`, location D3 and D1: the clone path takes the board branch's name, and the prime's `-weft` pairing, from the weft clone's checked-out `HEAD`. D3's records store has none, and the record names no other source.
   - CloneHub's step 3b reads "the weft primary's checked-out branch" from a full `git clone` of the weft and derives the Bolt branch and `<branch>-weft` from it (`internal/fabricengine/clone.go:304-326`, step list at `clone.go:131-138`).
   - D3 makes the records store with `git init`, the refspecs and a fetch, so nothing is checked out. Origin's `HEAD` points into `refs/heads/`, the code repository.
   - `remoteDefaultBranch` parses only `ref: refs/heads/` (`cleanupremotewarp.go:309-321`). D2's "a remote `HEAD` read for the records store answers the board branch" has no namespaced `HEAD` to read and no named branch to answer with.
   - D3's detection `ls-remote` for "the board branch under the namespace" needs that name before any store exists.
   - D1 or D3 has to fix the board branch's name in the namespace (for example, origin's code default branch name, or a constant) and the source the clone and the remote `HEAD` read take it from.

**Tool calls**

1. Searched fabric, gitrepo, hubreconcile and pairteardown for remote-config writes, plain clones, `Fetch` callers and remote `HEAD` reads, and read `warpDefaultBranch`. The manifest lacked these writers and readers.
2. Read `cloneRepo` (`clone.go:560-600`), its callers and `gitrepo.Repo.Fetch` (`internal/gitrepo/pull.go`). I needed whether the records store's fetch obeys the configured refspec.
3. Read `recordPush` (`internal/gitrepo/pushdestination.go`), the `ls-remote` form in `refscanner.go`, the push lines in `archive.go` and the head of `warplayout.go`. The manifest showed only the `branch.merge` line, not its writer.
4. Searched `reconcile.go` and `clone.go` for remote and layout reads, and counted the remaining `oneRepositoryAt` callers. I needed to know whether reconcile or `--reset` rewrites the records store's remote config.
