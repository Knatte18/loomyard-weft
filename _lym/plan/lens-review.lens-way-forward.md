```yaml
lens: way-forward
outcome: reported
```

Ten refusals and recovery steps checked. Eight progress from the state they leave; three findings below.

**Per refusal: next step, and whether it progresses**
- **Card 5, `lym-bolt` on origin, or a default branch tracking a root `.lym-anchor` or `_lym`:** delete `lym-bolt`, remove the markers, clone again. Progresses.
- **Card 5, `--single-repo` with the board ref present:** the same clone without the flag, which takes the join path. Progresses.
- **Card 5, `--single-repo` over a default branch holding `.lym-warp`:** a two-URL clone. Progresses.
- **Card 5, `--single-repo` with two URLs:** one URL. Progresses.
- **Card 5, seed records push or board push refused:** another host, or a two-repository hub. Progresses.
- **Card 5, board lease lost:** a one-URL clone joins the other hub. Progresses: the other hub's board is on origin, and the join adopts `origin/<prime>-weft` whatever the restore did.
- **Card 5, records lease lost or any other cause:** run the same clone again. Progresses: the rerun observes the new tip, or hits the board-present refusal, which leads to the join.
- **Card 8, unset `lym.primeRecordsBranch`:** `git -C <dir> config lym.primeRecordsBranch <prime-branch>-weft`. Progresses once the key is set.
- **Card 6, `--single-repo` with two URLs at the CLI:** the one-URL form. Progresses.
- **Card 6 / `CloneAndWire`, a failure before the seed push:** `lym fabric reconcile`. Does not progress correctly (finding 1).

**Findings**

1. **MEDIUM, cards 5 and 6 (D3 seed push):** a `--single-repo` clone that fails before `PushCloneBoard` leaves a hub whose way forward bypasses the leased, ordered seed.
   - `ReconcileHubWideAt`, the sparse-marker write or the Bolt commit can each fail inside `CloneAndWire` before the push step. `CloneAndWire` then returns `CloneResult{}`, so the `seed` mark is lost.
   - The hub stays intact, and `CloneAndWire`'s doc names `lym fabric reconcile` as the way forward.
   - Reconcile pushes the board with a plain `Bolt.PushRecorded` (`internal/fabriccli/fabric.go:700`, `internal/fabriccli/fabric.go:778`). That push is not leased on absence and does not push the prime's records branch first.
   - So the detection key can land before the anchor-carrying records branch. If another hub seeded meanwhile, `retryBoltPush` replays this board onto that hub's board instead of failing the lease.
   - Card 6 claims only "a seed push that fails has already removed the hub", and neither card names a way forward for this earlier failure.
   - Fix: tear the hub down on any `CloneAndWire` failure before the seed push for a `--single-repo` result, as the seed failure does. Or name `--reset` with the same clone as the way forward, and keep reconcile off an unseeded hub.
2. **LOW, card 5 (seed push cause classification):** a transient refusal gets a way forward that does not apply.
   - "Origin refused the push" is detected by `[remote rejected]` in stderr.
   - A transient `! [remote rejected] … (failed)` or `cannot lock ref` line matches it too. The plain push primitives retry exactly those lines (`internal/gitrepo#retryablePushRefusal`), but `UpdateRemoteBranchLeased` does not retry.
   - Such a refusal is told "use a host that accepts refs outside `refs/heads/`, or a two-repository hub". The step that would progress, running the same clone again, is not named.
   - Fix: classify a `(failed)` or `cannot lock ref` rejection as "any other cause".
3. **LOW, card 5 (seed refusal for a missing `--shortname`):** the card says the seed path requires the shortname "as a fresh bind's are", but not whether the refusal runs before the hub directory and the code clone exist.
   - The two-repository path resolves it before creation (`internal/fabricengine#CloneHub`, the `resolveEffectiveShortname` call ahead of `createExclusiveDir`).
   - If the seed path resolves it after cloning, the refusal costs a full clone and a teardown. Its way forward, adding `--shortname`, still progresses.
   - Fix: state that the check runs before anything is created, and add a row to `TestCloneHub_OneURLRefusals`.

Tool calls:
- One Bash call read three things my context lacked:
  - `resolveEffectiveWarpURL`'s return sites, to check the join's binding-mismatch refusal.
  - The `PushRecorded` call sites in `fabriccli`, to check what reconcile pushes on a left-intact hub.
  - Any settings rule that denies `git config`, for card 8's way forward. It found none.
