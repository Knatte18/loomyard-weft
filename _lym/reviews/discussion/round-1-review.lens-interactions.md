```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked every pair of decisions that act on the same state.

**Pairs checked**
- D1 × D2, the refspec as the layout key, with the fetch source under `refs/lym/heads/`: held.
- D1 × D4, the tag refspec against archive tags under `refs/lym/tags/archive/`: held.
- D1 × D9, the configured push refspec against the bare-push hazard: held. D2 forbids a bare push from fabric, and D9 documents the hazard.
- D2 × D4, delete of a pair's records branch by full namespaced name: broken (F1).
- D2 internally, the `gitrepo` parameter list against the upstream write and the remote delete: broken (F3).
- D2 × D3, the namespace and the remote `HEAD` read against the detection key, the "board branch": broken (F2).
- D3 × `CloneAndWire`, "seeds as a fresh two-repository bind" against "pushes before `CloneAndWire` runs": broken (F4).
- D3's refusal list × its acceptance criterion and Scope Out: held. The four refusals match one for one.
- D3 × D10, the f06acf39-origin refusal as the only code that knows that layout: held.
- D3 × D8, the builder "drives the clone of D3 by one URL": broken, NIT (F5).
- D5 × D6, removing the `protected` class against keeping `ProtectedBranchRefusal` non-retried: held. The criterion names the ordinary rejection class.
- D5 × D7, the removed probe and merge lifecycle against D7's deference to the peer: held.
- D5 × `## PATTERN changes`: held. The run-fsm text drops `protected` and `operator_unpushed`, and the cwd-resolution text matches D2's records root.
- D5 × the `PATTERN-test-only-code` constraint: held.
- D7 × its criterion, a cherry-pick in the records worktree: held whichever run lands first.
- D2 upstream × D1 fetch refspec: held. `branch.<b>.merge` set to `refs/lym/heads/<b>` maps through the fetch refspec to `origin/<b>`.
- D2's missing-store read × D3's init before first fetch: held.

**Findings**

**F1 — MEDIUM — design — `decision-record.md` D4 ("Teardown deletes the records branch by its full namespaced name")**
- **Summary:** D4's full-name delete breaks the deleted-or-absent distinction that the remote delete returns to fabric's mutation record. A missing ref then reads as deleted.
- **Evidence:** `internal/gitrepo/push.go:313`, whose comment says "a full refs/heads/ name deletes a missing ref silently". Its idempotent `(false, nil)` answer relies on `remoteRefAbsentTrigger` (`push.go:65`).
- **Reproduced:** with git 2.53.0, `git push <url> --delete refs/lym/heads/missing-weft` exited 0 and printed `[deleted]`.
- **Not covered elsewhere:** D2's namespace parameter covers push, lease and result matching, not this case. The callers are `internal/fabricengine/destroy.go:1631` and `:1643`.
- **Fix:** D4 states how an absent namespaced ref is told apart, such as an `ls-remote` first or a leased delete. It also records the observed result.

**F2 — MEDIUM — design — `decision-record.md` D2 ("A remote `HEAD` read … answers the board branch") and D3 (the detection key)**
- **Summary:** D2 and D3 both turn on "the board branch" under `refs/lym/heads/`, but no decision fixes that branch's name.
- **Today's source of the name:** a two-repository hub takes the board branch from the weft origin's `HEAD`, through `remoteDefaultBranch` (`internal/fabricengine/cleanupremotewarp.go:308-325`, called from `addadopt.go:68` and `cleanupremotewarp.go:87`).
- **The gap:** a records store made by init and fetch has no namespaced `HEAD`. D3's one-URL `ls-remote` and D2's `HEAD` answer each need a name, which Plan would have to invent independently.
- **Fix:** name the board branch and the prime's records branch in D1, for example as derived from the code origin's default branch. Say how D2's `HEAD` answer obtains it.

**F3 — LOW — consistency — `decision-record.md` D2 ("`gitrepo` takes the namespace as a parameter of its push, lease and push-result matching")**
- **Summary:** D2's own list of what `gitrepo` takes leaves out two `gitrepo` operations whose ref names it does send to origin:
  - the upstream config, which D2 assigns to fabric but `gitrepo` writes as `refs/heads/`;
  - the remote delete.
- **Evidence:** `internal/gitrepo/pushdestination.go:114` writes `branch.<b>.merge` as `refs/heads/<b>`. `internal/gitrepo/push.go:314` and `:345-346` delete, and the lease delete hard-codes `refs/heads/`.
- **Fix:** widen the parameter sentence to every `gitrepo` operation that sends or writes an origin ref name, including the upstream write and remote delete.

**F4 — MEDIUM — consistency — `decision-record.md` D3 ("seeds the records store as a fresh two-repository bind seeds its weft" / "Before `CloneAndWire` runs, it pushes the seed")**
- **Summary:** the two halves of D3 contradict each other and the code they reuse.
- **How the reused path pushes:** a fresh two-repository bind pushes nothing in `CloneHub`. Its board push runs inside `CloneAndWire`, after the config commit (`internal/fabriccli/clone.go:73-96`). Its prime records branch is not pushed at clone, "leaves the upstream to the first push" (`internal/fabricengine/clone.go:513`).
- **Why "before `CloneAndWire` runs" fails:** `CloneAndWire` is the entry that calls `CloneHub` (`internal/fabriccli/clone.go:37`), so nothing it runs can come before it.
- **What D3 adds:** pushing the prime's records branch at clone is new behaviour, not the reused seeding path its rationale claims.
- **Fix:** state where in the sequence the seed push runs, such as at the end of `CloneHub` before the config commit, and what the seed holds at that point. Say that it is added behaviour, not the reused bind path.

**F5 — NIT — consistency — `decision-record.md` D8 ("drives the clone of D3 over it by one URL")**
- **Summary:** over a bare origin with no namespace, D3 sends a plain one-URL clone down the two-repository path. The builder must therefore use `--single-repo`, which D8 leaves unsaid.
- **Evidence:** D3's paragraph "One URL whose origin holds no namespaced board branch takes today's two-repository path".
- **Fix:** D8 names `--single-repo` for the first clone and a plain one-URL clone for a second machine.

**Out of lens (noted only):** D3's bound "force-replaces only … while no board branch exists" is checked before anything is created but enforced at push time. That is a race for the bound-breaker lens.

**Tool calls:** none. My context already held every file read.
