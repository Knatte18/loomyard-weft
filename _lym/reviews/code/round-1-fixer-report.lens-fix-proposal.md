```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: challenge the reviewer's proposals the fixer carried out (F1–F5).

**Findings**

1. **MEDIUM** — F1, `internal/fabricengine/remoteleftover.go` (`originRef` and `Error()`). The proposal fixes only the message text, and a twin of that message was left as it was.
   - `internal/battencli/readoptstop.go:195` builds its own way-forward for the same weft leftover. It prints `git -C <recordsRoot> push --force-with-lease=refs/heads/<leftover.Branch>:<tip> origin --delete <leftover.Branch>`.
   - In a one-repository hub, `refs/heads/<short>` on origin is not the leftover. The leftover is `refs/lym/heads/<short>`, so both the lease and the delete name the wrong ref.
   - The case: a leftover refusal for a weft branch in a one-repository hub reaches `leftoverArchiveWayForward`. The operator gets a command that cannot reach the ref, which is the premise F1 raised.
   - The fixer changed `Error()` and left this site.
   - `originRef` is unexported, so `battencli` cannot read it. That is why the twin stayed stale.

2. **LOW** — F1, `internal/fabricengine/remoteleftover.go` (`Error()`, weft case). The weft message now depends on a field that only one constructor sets.
   - The zero `kind` is `leftoverDivergedAdoptedWeft`, so any `ErrRemoteLeftover` built without `originRef` prints `--delete ` with an empty ref.
   - `internal/battencli/owner_readopt_test.go:135` builds one that way, with no `kind` and no `originRef`. `owner_readopt_test.go:119` reads `addErr.Error()`, though what that error holds is not visible from here.
   - The one production weft construction does set the field, so no shipped path is affected. The case is a test or external construction.

3. **LOW** — F3, `internal/fabricengine/destroy.go` (`pushCloneSeed` upstream loop). The new message says to run `lym fabric clone <url>` to join the seeded hub.
   - This failure path returns with the hub left in place.
   - A plain clone may refuse to run next to that existing hub, or may need the container removed first. I did not verify whether `lym fabric clone` handles an existing container.
   - The hand-set `branch.main.*` alternative in the same message is sound.

No findings for F2, F4 or F5.
- F2: the identity case is `refs/tags/T:refs/tags/T` and the namespaced case is `refs/tags/T:refs/lym/tags/T`. Both are valid pushes.
- F4: the new rows pin the namespaced tag mapping.
- F5: the reworded bullet is consistent with the text under it.

**Tool calls**
1. `grep` over `internal/battencli` tests and `readoptstop.go` for `Error()` and leftover uses: to see how `battencli` consumes `ErrRemoteLeftover`, which my context lacked.
