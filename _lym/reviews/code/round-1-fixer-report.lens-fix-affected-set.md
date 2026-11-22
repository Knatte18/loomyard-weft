Task: check that the fix covers every member of the sets behind F1 (Fetch prune docs, callers, tests) and F3 (re-judge callers).

No findings of BLOCKING or MEDIUM severity. One LOW:

- **LOW — `internal/gitrepo/pull.go` (`fetchRemote = "origin"`):** the prune pass reads refspecs for `origin` only. The plain `git fetch` before it follows whatever remote the branch tracks. A checkout whose tracked remote has another name still fetches, but its deleted branches are never pruned. Evidence: `RemoteFetchRefspecs(fetchRemote)` returns `ErrRemoteNotConfigured` there, so `Fetch` returns nil early. The review's fix named no remote. I did not search the repo for non-`origin` remotes, so I can't say whether any real store has one.

F1 set checks:
- A grep for `--prune` and prune wording across Go and markdown, excluding `_lym`, `.lym` and `.scratch`, found no other place that claims bare-prune behaviour.
- `internal/gitrepo/doc.go:42`, the `pull.go` header and the `Fetch` comment all now say prune covers tracking refs only. The `Fetch` comment's "`git fetch`" wording is accurate.
- `internal/battencli/arm.go:651` still says "the fetch prunes such a branch's ref". That is accurate for tracking refs.
- All `Fetch()` callers in `fabricengine` and `battencli` use the same method, so none needs a signature change. The listed sites include `clone.go:845`, `pushclassified.go`, `handover.go`, `takeoverwrite.go`, `mergeguards.go`, `parenttips.go`, `merge.go:567`, `destroy.go:1747`, `codetipsettle.go`, `bolt.go:313`, `catchup.go:224` and `arm.go:641`.

F3 set checks:
- `hubRunVerdict` in `battencli` is called only at `arm.go:345`. The fetch closure there now also fetches the prime's code store.
- `internal/loomcli/ownercheck.go:142` has its own `hubRunVerdict`, wired at `loomcli/cli.go:209`. I did not open it, so I can't say whether it judges a code-tip wait. It is outside this change set.

Tool calls: 2 Bash greps, one for the prune wording and one for the `Fetch` and `hubRunVerdict` callers. My context lacked a repo-wide caller and doc search.
