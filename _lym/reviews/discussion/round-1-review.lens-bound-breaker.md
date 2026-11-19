```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked each bound, ordering promise and existing-code claim in the decision record.

**Claims attacked**
- D3, the refusal of an origin "whose default branch tracks `.lym-anchor` at its root", applied to every one-URL clone: **broken** (F1).
- D3's bound that it "force-replaces only a namespaced records branch while no board branch exists": **broken** (F2).
- D4's teardown delete "by its full namespaced name", and the git evidence behind it: **broken** (F3).
- D3's detection key, "the board branch under the namespace", and D2's remote `HEAD` read answering the board branch: **broken** (F4).
- D3's claim that "a refused push leaves nothing on origin that reads as a hub, and nothing on disk": **partly broken** (F5).
- D3's claim that it "writes no commit to any code branch": held. `CloneAndWire` commits the module configs through the anchored location, which is the records worktree under two-repository geometry.
- D1's claim that the code store "never fetches or pushes a records ref" and the records store "never holds a code ref": held. Default refspecs skip `refs/lym/`, and tag auto-follow brings nothing because records history holds no code commit.
- D2's bound that "a misread fails closed, except for a missing store": held. Remote operations run in the store directory, such as `weftRoot` in `remoteleftover.go`, and that directory exists in every hub that has a store.
- D2's claim that "no records-store path reads origin's `refs/heads/`": held as a stated rule. The `ls-remote --heads` and `--symref HEAD` sites are covered by D2's list.
- D4/acceptance, "add no ref under `refs/tags/` beyond the code branch": held at two-repository parity. A discard pushes the code-tip archive tag to the code origin in both layouts (`remoteteardown.go:258-260`).
- D5/D6, the protected refusal reclassified as the ordinary rejection class: held. It matches the pre-f06acf39 behaviour.
- D7's conditional on the peer run landing first: held as written. Ordering against the peer is the parent's question under Open risks.

**Findings**

**F1 — BLOCKING · design · `_lym/discussion/decision-record.md` D3, refusal list (line 117)**
The root `.lym-anchor` refusal also catches a two-repository weft whose board still carries the legacy hub-wide marker.
- **Counterexample:** a one-URL clone of an existing two-repository hub's weft URL, where that hub was never reconciled.
  The board, `weft:main`, is the weft's default branch and still tracks `.lym-anchor` at its root.
  The origin has no namespaced board, so the record sends it down today's two-repository path, but the refusal applies to "every one-URL clone" and refuses it.
- **Harm:** this contradicts Scope Out ("no behaviour change to a two-repository hub").
  The way forward also tells the operator to remove `.lym-anchor` and `_lym` from "the code branches", which here are the records.
- **Evidence:**
  - `internal/fabricengine/anchormarker.go:2`: "the legacy hub-wide marker sits on the board".
  - `internal/fabricengine/clone.go`, `CloneHub` doc step 5: "else the Bolt legacy marker on weft:main".
  - `internal/fabricengine/recordsprobe.go`, `probeWeftBinding`: `probeTreeHasPath(HEAD, .lym-anchor)`.
- **Fix:** scope the `.lym-anchor` arm to an origin that also lacks a `.lym-warp` binding, or to `--single-repo`, or drop it and keep only the `lym-bolt` arm.

**F2 — MEDIUM · design · D3, bound and leftover replacement (lines 105–109, 120–121)**
"While no board branch exists" is checked before anything is created, not when the push runs, so the force-replace is unbounded at push time.
- **Counterexample:** machine A's `--single-repo` has pushed `refs/lym/heads/<prime>-weft` but not yet its board.
  Machine B's `--single-repo` passes its precondition (no board) and force-replaces A's records branch.
  A's board push then lands. B's board push is refused, and B tears down.
- **End state:** origin holds A's board beside B's prime records branch, and A's local records branch has diverged from origin.
- **Fix:** lease the records push on the tip, or the absence, observed at the precondition, and lease the board push on absence (`--force-with-lease=refs/lym/heads/<board>:`).

**F3 — MEDIUM · design · D4, teardown delete (lines 133–134)**
Deleting by the full namespaced name breaks `gitrepo.DeleteRemoteBranch`'s absent-ref contract, and D4's own evidence does not model the code path.
- `DeleteRemoteBranch` uses the short name on purpose: "a full refs/heads/ name deletes a missing ref silently". It returns `(false, nil)` for an absent ref (`internal/gitrepo/push.go:299-324`).
- Under git 2.53.0 in a scratch repository, `git push <url> --delete refs/lym/heads/missing-weft` exited 0 and printed `remote: warning: deleting a non-existent ref` and `- [deleted]`.
  A teardown of a records branch that was never pushed would therefore record a removal that never happened.
- D4's evidence ran `git push origin --delete <branch>` under the mapped push refspec.
  `gitrepo` pushes to `dest.url`, where `remote.origin.push` is not applied (`push.go:314`).
- **Fix:** state how the namespaced delete tells an absent ref apart, for example an `ls-remote` of the full ref first, or a lease on the observed tip.

**F4 — MEDIUM · design · D1/D3 detection key, D2 remote `HEAD` (lines 79, 96–97, 105–106)**
No decision names the board branch inside the namespace, yet it is the layout's detection key.
- Today the board is the weft's default branch, found through origin's `HEAD` (`ls-remote --symref origin HEAD`, `internal/fabricengine/cleanupremotewarp.go:308-325`). The namespace has no `HEAD`.
- **Counterexample:** if the name is taken from the code repository's default branch, an operator who renames that branch on origin (`main` to `trunk`) makes the one-URL probe miss `refs/lym/heads/main`.
  `--single-repo` then passes its precondition and seeds a second hub beside the first, in the same namespace.
- **Fix:** fix the board branch's name in D1 as a constant, independent of origin's `HEAD`, and say how the clone and the remote-`HEAD` read name it.

**F5 — LOW · design · D3, seed-push failure (lines 107–109, 123–125)**
The teardown and "nothing on disk" promise names only a push that origin refuses.
A transport failure after the records push and before the board push is not named, so the hub directory's fate in that case is unstated.
- **Fix:** say that any failed seed push, a refusal or a transport error, takes the same teardown path.

**Tool calls**
- One grep and read of `internal/fabricengine/anchormarker.go` (legacy marker) and `internal/fabricengine/remoteteardown.go:225-280` (teardown tag pushes and ways forward). My context had neither the legacy board-marker placement nor the code-tip archive tag push path.
