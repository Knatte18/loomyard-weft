---
round: 1
focus:
  - "internal/fabricengine/clone.go, destroy.go: does the --single-repo seed push order the prime's records branch (leased on the observed tip or absence) before the board branch (leased on absence), and does a failed push remove the hub and restore the records branch to the observed state, leased on the pushed tip?"
  - "internal/fabricengine/clone.go refuseOneURLClone: does every D3 refusal run before anything is created, from one ls-remote, and name its way forward (namespaced board with --single-repo, .lym-warp on the default branch, second URL, lym-bolt, root .lym-anchor or _lym)?"
  - "internal/gitrepo push.go, pushretry.go, refreads.go, remote.go, blobread.go, gogit.go: is the namespace passed to every operation that sends or writes an origin ref name (push, lease, push-result matching, upstream merge write, plain and leased remote delete), with ls-remote results mapped back and no --heads or --tags selection on a records store?"
  - "internal/fabricengine/destroy.go plain remote delete: does it first ls-remote the full namespaced name in both layouts, so a never-pushed branch records no removal?"
  - "internal/fabricengine/recordslayout.go: is the layout read from the records store's fetch refspec, cached only for an existing store, quiet and uncached for a missing store, and an error (never a two-repository answer) when the remote config cannot be read?"
  - "internal/gitrepo/gogit.go tolerant config helper: do Upstream, UpstreamSHA, resolvePushDestination, RemoteURL and RemoteFetchRefspecs all read through it, and do the namespaced branch.<b>.merge writes stay git-valid?"
  - "primaryWeftBranch and resolveManagedBranch (weftbranch.go, cleanup.go, checkout.go): does a one-repository hub read lym.primeRecordsBranch, fail closed with the named way forward when the key is missing, and keep the prime's records branch out of the orphan sweep after a checkout moves the prime?"
  - "Fabric constants for refs/lym/, heads/, tags/, board branch main, lym-bolt and the config key: is any of them a literal at a use site in the diff, against the PATTERN-code-conventions change?"
  - "Scope: does the diff leave a layout predicate or namespace read outside fabricengine and gitrepo (fabriccli, lymcwd, battencli, loomcli, hubreconcile), or a code-store ref touched by the translation?"
  - "Tests (clone_singlerepo, onerepository_lifecycle, recordslayout, cleanup, gitrepo push tests): do they assert independent literals rather than recompute the code's own translation, and does each D3 refusal and the acceptance criteria's master-default and second-machine cases have coverage?"
  - "Docs and PATTERN: do README.md, docs/getting-started.md, docs/overview.md and the PATTERN.md and pattern/ diff land the D9 hazards and the declared PATTERN-code-conventions change and PATTERN-single-repo-hub add, with no leftover records-on-code-branch or lym-bolt text?"
---

Focus covers the decision record's D1 to D4 mechanics, where a missed namespace site breaks a one-repository hub silently,
plus the declared PATTERN and doc landings.
