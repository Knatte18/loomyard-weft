```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: each decision's mechanism run end to end on one concrete case, checking the end state against the decision's purpose.

**Decisions walked:**
- **D1:** a `--single-repo` hub over a code origin whose prime is `main`. The board is named after the code prime's checked-out branch (`clone.go` `checkedOutBranch` → `ensureBoardWorktree`), so it lands at `refs/lym/heads/main` and the prime's records at `refs/lym/heads/main-weft`. Placement holds. The board's name is stated nowhere (see D3 detection).
- **D2:** a pair push of `t-weft` through `pushPlainWithRetry`, then `--track` upstream and a `git status` sync read. Met, given the namespace parameter. `--track` derives `branch.<b>.merge` through the fetch refspec, so the stated upstream holds.
- **D3, fresh `--single-repo`:** not met; the seed push cannot run before `CloneAndWire` (F1).
- **D3, rerun after a partial push:** not met; the leftover records branch is adopted, not replaced (F2).
- **D3, second-machine one-URL clone:** met only while origin's `HEAD` still names the branch the board was seeded under (F3).
- **D4:** teardown of `t-weft`. Not met for the absent-ref case: the full-name delete reports a removal that never happened (F4).
- **D5, D6:** a protected push over the remaining classification. Met.
- **D7:** a status commit during a weft cherry-pick. Met.
- **D8:** the builder drives the D3 clone, so it inherits F1.
- **D9, D10:** prose and a refusal. Nothing to execute; met.

**Findings**

**F1. BLOCKING (design), D3 `--single-repo` seed order.** The seed push "before `CloneAndWire` runs" has no board commit to push.
- At the end of `CloneHub` on a fresh bind, the board is an unborn orphan branch (`internal/fabricengine/boardweft.go:55-57`).
- `.lym-warp` and `.lym-shortname` are only written into the board dir. Per the comment at `internal/fabricengine/clone.go` ~395-418, they are "committed by the same CLI-layer Bolt commit", which is `internal/fabriccli/clone.go:74`, inside `CloneAndWire`.
- `CloneAndWire` also calls `CloneHub` itself (`internal/fabriccli/clone.go:37`), so nothing can run before it.
- On the two-repository path D3 says the clone reuses, `CloneAndWire` pushes the board first (`internal/fabriccli/clone.go:92-96`). That pushes the detection key before the prime's records branch, and a refused push there leaves the clone intact (doc comment at line 34).
- The record-first, board-last, teardown-on-refusal order needs a seed push after the Bolt commit and a layout branch in `fabriccli`. That is exactly what f06acf39's `res.OneRepository` / `PushOneRepositoryClone` sequence is (`internal/fabriccli/clone.go:90-96,139-145`; `internal/fabricengine/clone_onerepository.go:304-322`). D5 removes it, and the Goal ("every other module sees a two-repository hub") forbids it.
- Fix: D3 should state where the seed push sits relative to the Bolt commit, and how `fabriccli` reaches it without reading the layout.

**F2. MEDIUM (design), D3 rerun after a partial push.** A leftover namespaced records branch is adopted, not "replaced by the seed push".
- The records store fetches with D1's refspec, so the leftover arrives as `origin/main-weft`.
- `suffixWeftPrimaryBranch` then checks out from it (`internal/fabricengine/clone.go:515-524`).
- The anchor adopt path reads the leftover's committed `.lym-anchor` (`clone.go` ~368).
- A rerun with a corrected `--subpath` is refused by `adoptRecordedAnchor`: "requested --subpath … does not match the recorded anchor" (`clone.go:478-479`). That message names no way forward, and the leftover never gets force-replaced.
- So the force-replace that D3's bound describes never happens on this path, and the "rerun succeeds" criterion holds only for an unchanged subpath.
- Fix: decide whether the rerun skips fetching or adopting the leftover, or adopts it, and state which.

**F3. MEDIUM (design), D3 one-URL detection and D1's board name.** The detection key is "the board branch under the namespace", but no decision names the board branch.
- The board is named after the code prime's checked-out branch (`clone.go` `checkedOutBranch(warpWorktreePath…)` → `ensureBoardWorktree`).
- Detection runs before anything is cloned, so its only source for that name is origin's code `HEAD`.
- If the code repository's default branch changes after seeding, a second machine looks for `refs/lym/heads/<new>` and misses it.
  - Without `--single-repo`, the URL is read as a weft.
  - With `--single-repo`, a second hub is seeded over the first, passing the "origin holding the namespaced board branch" refusal.
- The same name feeds D2's "a remote `HEAD` read answers the board branch".
- Fix: D1 should name how the board branch is chosen and found.

**F4. MEDIUM (design), D4 teardown "by its full namespaced name".** This reaches `gitrepo.DeleteRemoteBranch`, whose deleted/absent answer rests on the short-name form.
- The comment at `internal/gitrepo/push.go:313` says: "a full refs/heads/ name deletes a missing ref silently".
- The handler's git 2.53.0 run confirms it: `git push <url> --delete refs/lym/heads/missing-weft` exits 0 and prints `[deleted]`.
- An absent records branch therefore returns `deleted=true`, and the mutation record in `destroy.go:1631-1643` claims a removal that never happened.
- D4's own evidence, `git push origin --delete` under a mapped push refspec, also does not model the code: `gitrepo` pushes to `dest.url`, where no configured push refspec applies (`push.go:314`).
- Fix: state how teardown tells an absent namespaced branch apart, for example by a pre-delete `ls-remote` or the leased delete.

**Tool calls:**
1. Read `internal/fabricengine/boardweft.go` 1-60 and `clone.go` 260-420. The manifest lacked how `CloneHub` names the board branch and where the fresh bind commits `.lym-warp`.
2. Read `suffixWeftPrimaryBranch` and `adoptRecordedAnchor` in `clone.go`, and `PushOneRepositoryClone` in `clone_onerepository.go`. The manifest lacked the weft-primary adopt step and the current seed-push placement that D3's rerun and order run through.
