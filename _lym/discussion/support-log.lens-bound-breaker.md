```yaml
lens: bound-breaker
outcome: reported
```

Lens bound-breaker: I attacked each bound, guard and code claim in the decision record and support log. Two claims broke as BLOCKING, three as MEDIUM, and two minor gaps are LOW.

**Findings**

1. **BLOCKING — D2 and Constraints (`PATTERN-gitrepo-client-boundary`).** The ref-name translation is placed in fabric only, but `gitrepo` spells the remote ref itself, so records pushes would bypass it.
   - Evidence: `internal/gitrepo/push.go` `pushPlainWithRetry` runs `git push <dest.url> <source>:refs/heads/<branch>`, and `UpdateRemoteBranchLeased` runs `--force-with-lease=refs/heads/<branch>:<sha>` with `<sha>:refs/heads/<branch>`.
   - Both push to the resolved URL with an explicit `refs/heads/` destination. That bypasses the records store's `remote.origin.push` mapping and any fabric-side translation.
   - Counterexample: in a one-repository hub, a pair's `x-weft` records push creates a visible branch `refs/heads/x-weft` on origin.
   - A board push of `main` targets code `refs/heads/main`. It is rejected only because the histories are unrelated; the leased path compares against the wrong ref.
   - The decision has to state that the translation boundary covers `gitrepo`'s push destination, lease ref and landed-push tracking.

2. **MEDIUM — D2 failure direction.** The layout read "reads a path with no store on disk as two repositories" and caches the answer per process. Two repositories means the identity translation, so any misread fails open: records go into `refs/heads/` on a one-repository origin.
   - Counterexample: a long-lived process, or the clone itself, reads a path before its records store exists. It caches "two", and a later push from the new store goes untranslated.
   - The record should bound this. Cache no answer for a missing store, and treat a store whose refspec cannot be read as an error rather than as two repositories.

3. **MEDIUM — D7 deferred mail.** Both the bullet and its acceptance criterion describe the behaviour f06acf39 left in place, so neither tests a revert.
   - Evidence: `internal/fabricengine/failedpush.go` line 174 currently skips mail when `Class == Deferred && len(Commits) == 0`.
   - Exploration reported that before f06acf39 every deferred record was skipped. The widening is that a deferred record with commits is now mailed.
   - "A deferred failed-push record with no commits is not mailed" holds both before and after. The criterion should be that a deferred record naming commits sends no mail.

4. **MEDIUM — D10 and D3 over an origin f06acf39 already used.** The way forward "is cloned again with D3" breaks on an origin where f06acf39's `--single-repo` pushed its marker commit.
   - That origin has no `refs/lym/` namespace, and `.lym-warp` sits only on `lym-bolt`, not on the default branch. So all of D3's `--single-repo` refusals pass.
   - The code branch still carries the tracked `.lym-anchor` and `_lym`. Junction wiring then meets a tracked real `_lym` directory in the prime (`seedLymJunction` checks for a link and errors or re-points).
   - The stale `lym-bolt` branch stays visible on origin.
   - Either refuse an origin that holds `lym-bolt` or a root `.lym-anchor` and name the cleanup, or state the manual cleanup in D10.

5. **MEDIUM — D1 and D3, records store creation.** "A clone of the same URL with D1's refspecs" does not say how the store is made, and today's `cloneRepo` is a plain `git clone`.
   - A plain clone fetches every code branch and object into the records store. It also leaves `refs/remotes/origin/main` as code `main`, which collides with the board's tracking ref `origin/main` under D1's fetch mapping.
   - The record bounds the code store ("never fetches a records ref") but not the reverse. It should state that the records store is configured before its first fetch and never holds a code ref.

6. **LOW — D3 and D9.** `--single-repo` on the code repository of an existing two-repository hub passes every refusal: no namespace, and no `.lym-warp` on its default branch.
   - It silently starts a separate hub. f06acf39's docs named this hazard, and D9's hazard list drops it.

7. **LOW — D4 remote cleanup.** `cleanupremotewarp.go` takes `remoteBranchTips` and `remoteDefaultBranch` (`ls-remote --symref origin HEAD`) from a repo directory and archive tags from the weft.
   - In a one-repository hub, any weft-side `ls-remote` of `HEAD` returns the code default branch. The record does not cover remote-HEAD reads in its translation list.
   - Not verified beyond the call sites.

**Claims attacked**

- D1: the code store never fetches records refs — held, given git's default refspecs.
- D1: GitHub accepts pushes to `refs/lym/` — held as evidence, and the D3 refusal covers a host that rejects them.
- D2: every ref name fabric spells passes the translation — broken (finding 1).
- D2: the cached, quiet read — broken in its failure direction (finding 2).
- D2: no module outside fabric knows the layout — held, but `gitrepo` must become namespace-aware (finding 1).
- D3: `--single-repo` writes only under `refs/lym/` — broken through finding 1's push path. It holds once that is fixed.
- D3: the refusals catch a reused origin — broken for an f06acf39 origin (finding 4) and a two-repository code origin (finding 6).
- D3: a refused push leaves no hub directory — held.
- D4: one code path per operation — held, subject to findings 1 and 7.
- D5 and D6: the push-class keep and remove split — held.
- D7: merge probe and commit refusal order — held.
- D7: deferred mail — broken (finding 3).
- D10: re-clone needs no conversion — broken (finding 4).
- Open risk on the peer run's overlapping files — held.

**Tool calls made**

- Grepped `internal/fabricengine/failedpush.go` for `Deferred` and fabric's remote-HEAD reads, to verify the deferred-mail rule and the default-branch reads my context lacked.
- Read the `cleanupremotewarp.go` span around `remoteDefaultBranch`, and grepped for explicit `refs/heads` refspecs in `fabricengine` and `gitrepo`. My context did not show which repository these act on or where refspecs are spelled.
- Read the `internal/gitrepo/push.go` spans for `pushPlainWithRetry` and `UpdateRemoteBranchLeased`, to see how the push destination and lease ref are formed.
- Grepped `seedLymJunction` in `internal/fabricengine/junction.go`, to check how wiring treats an existing `_lym` for finding 4.
